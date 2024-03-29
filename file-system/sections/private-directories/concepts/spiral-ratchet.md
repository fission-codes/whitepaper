---
description: Backwards-secret versioning
---

# Spiral Ratchet

Every node in the private tree is encrypted with a different key. This is done randomly for child nodes \(along the y-axis\), and deterministically with a cryptographic ratchet for increasing versions \(along the z-axis\).

The basic idea for cryptographic ratchets is that repeatedly hashing a value creates a kind of backwards-secret clock. When you start watching the clock, you can generate the hash for any arbitrary future steps, but not steps from prior to observation since that requires computing the SHA preimage.

SHA-256 is native to the WebCypto API, is a very fast operation, and commonly hardware accelerated. Anecdotally, Firefox on an Apple M1 completes each SHA ~30μs \(10k/300ms\). The problem with a single hash counter is threefold:

1. The root of the unencrypted tree updates with with every atomic operation, and thus accrues a lot of changes  
2. An actor may be many months since their last sync, and need to fast forward their clock by some huge number of elements  
3. Seeking ahead by 100ks or millions takes very noticeable time  
4. It should be cheaper to fast forward than for an attacker to build a large history

We still want small step changes to very fast, but also be able to deterministically tune how quickly we jump ahead, without revealing previous counter hashes, while maintaining the same security properties as a single ratchet counter.

Positional counting does exactly this in the most common numeral systems. Our design involves three positions, two of which are bounded. We call these the large \(or epoch\), medium, and small numbers. 

Positionally this look like:

```text
large   medium  small
l*2^16  m*2^8   s*2^0

l = 0x600b56e66b7d12e08fd58544d7c811db0063d7aa467a1f6be39990fed0ca5b33
m = 0x5d58264a09dce1f2676e729d0ea1db4bf90b9be463d7fc1aa9b43b358e514599
s = 0xc8633540cabdf591e07918a2595964cc1b692d0f9392f079f2f110c08b67c6f4
```

The large and small are bounded at 256 elements. We achieve this by keeping a record of the max bound of these numbers. This is found by hashing that number 255 times \(0 is the unchanged value\). As we increment each number by taking its SHA256, we check if it matches the max bound. If it does, we increment the next-highest value, and reset the smaller values.

The metaphor is a spiral. You can ratchet one at a time, or deterministically skip to the start of the next ring in the spiral.

![https://commons.wikimedia.org/wiki/File:Ulam\_spiral\_howto\_all\_numbers.svg](../../../../.gitbook/assets/532px-ulam_spiral_howto_all_numbers.svg.png)

```haskell
data SpiralRatchet = SpiralRatchet
  { large     :: SHA256

  , medium    :: SHA256
  , mediumMax :: SHA256
  
  , small     :: SHA256
  , smallMax  :: SHA256
  }
  
exampleSR = SpiralRatchet
  { large     = 0x600b56e66b7d12e08fd58544d7c811db0063d7aa467a1f6be39990fed0ca5b33
  
  , medium    = 0x5d58264a09dce1f2676e729d0ea1db4bf90b9be463d7fc1aa9b43b358e514599
  , mediumMax = 0x5aae7b2b881d21863292a1556eafd2a3b21527f64f33c6fcc2beaa9d9cf1fe5f
  
  , small     = 0xc8633540cabdf591e07918a2595964cc1b692d0f9392f079f2f110c08b67c6f4
  , smallMax  = 0xb95c5d8851daff6204eb227f56d8c6af1c11a80d46d17eb0aa219a9d2ec109af
  }
```

Incrementing a larger value resets all smaller values. This is done by taking the SHA-256 of the \(one's\) complement. This cascades from larger to all smaller positions.

{% hint style="danger" %}
Please note that JavaScript's basic inverse function behaves unexpectedly. The `~` operator casts values to their one's compliment _integer_. This means that `~0b11 !== 0b00`, but rather `~0b11 === -4`. `-4` can not be directly expressed in JS binary, since it interprets binary notation as being a natural number only \(rather than its two's compliment\). To keep this from happening, use a toggle mask of equal length: `val ^ b1111...`.
{% endhint %}

The small and medium values are base-256. This means that the first position increments in ones \(2^0\), the second position increments in 256s \(2^8\), and the third position by 65,536s \(2^16\). Looking at a triple does not admit which number it is, only which number relative to the max bounds of the medium and small numbers. It is not possible to determine which of epoch \(large number\) you are in. Here is an example of a spiral ratchet being constructed from a seed value.

```haskell
seed       = 0x600b56e66b7d12e08fd58544d7c811db0063d7aa467a1f6be39990fed0ca5b33
large      = sha256 seed -- e.g. 0x8e2023cc8b9b279c5f6eb03938abf935dde93be9bfdc006a0f570535fda82ef8
  
mediumSeed = sha256 $ Binary.complement seed
medium     = sha256 mediumSeed
mediumMax  = iterate sha256 medium !! 255
   
smallSeed  = sha256 $ Binary.complement mediumSeed
small      = sha256 smallSeed
smallMax   = iterate sha256 small !! 256
```

Here is how to increment the ratchet by one:

```haskell
-- large (0..), medium (0-255), small (0-255)

toVersionHash :: SHA256
toVersionHash SpiralRatchet {..} = sha256 (large `xor` medium `xor` small)

advance :: SpiralRatchet -> SpiralRatchet
advance SpiralRatchet {..} =
  case (nextSmall == smallCeiling, nextMedium == mediumCeiling) of
    (False, _)    -> advanceSmall
    (True, False) -> advanceMedium
    (_,    True)  -> advanceLarge
    (False, True) -> error "Not possible"

  where
    nextSmall  = sha256 small
    nextMedium = sha256 medium

    advanceSmall :: SpiralRatchet
    advanceSmall = SpiralRatchet {small = nextSmall, ..}

    advanceMedium :: SpiralRatchet
    advanceMedium =
      let
        resetSmall  = sha256 $ compliment nextMedium
        resetMedium = sha256 $ compliment nextLarge

      in
        SpiralRatchet
          { large      = large

          , medium     = nextMedium
          , mediumCeil = recursiveSHA 256 nextMedium

          , small      = resetSmall
          , smallCeil  = recursiveSHA 256 resetSmall
          }

    advanceLarge :: SpiralRatchet
    advanceLarge =
      let
        nextLarge   = sha256 large
        resetMedium = sha256 $ compliment nextLarge
        resetSmall  = sha256 $ compliment resetMedium

      in
        SpiralRatchet
          { large      = nextLarge

          , medium     = resetMedium
          , mediumCeil = recursiveSHA 256 resetMedium

          , small      = resetSmall
          , smallCeil  = recursiveSHA 256 resetSmall
          }
```

Many files will have few revisions. To prevent leaking this information, we take advantage of the fact that the setup starts at a random point in the ratchet:

```haskell
setup :: SpiralRatchet
setup = do
  largePre   <- random -- ratchet seed
  mediumSkip <- randomBetween 0 254
  smallSkip  <- randomBetween 0 254

  let
    large = sha256 largePre

    mediumPre = sha256 $ compliment largePre
    medium    = recursiveSHA mediumSkip mediumPre
    mediumMax = recursiveSHA 255 mediumPre

    smallPre = sha256 $ compliment mediumPre
    small    = recursiveSHA smallSkip smallPre
    smallMax = recursiveSHA 255 smallPre

  return SpiralRatchet {..}
```



