```haskell
module Exercise05 where

import ClassyPrelude
import Data.Aeson (decode, encode)
import Test.Hspec (Spec, describe, shouldBe, shouldSatisfy)
import Test.Hspec.QuickCheck (prop)
import Test.QuickCheck (Arbitrary, arbitrary, elements)

import Exercise03
```

# Exercise 05

It worked! Now, let's write some tests so that we can do this once and we never have to think about it ever again, even
when we refactor. We create a new enumeration, generate the instances, and test the instances. That way we're testing
that (1) the template haskell even compiles, because that would be terrible if it didn't, and (2) that the generated
code does what we want whenever we generate the way we expect.

```haskell
data Fantastic
  = FantasticMrFox
  | FantasticBeasts
  | FantasticFantasia
  deriving (Eq, Show)

data Incorrect
  = Correct
  deriving (Eq, Show)

deriveEnumInstances ''Fantastic
-- deriveEnumInstances ''Incorrect -- should fail

instance Arbitrary Fantastic where
  arbitrary = elements [FantasticMrFox, FantasticBeasts, FantasticFantasia]

thEnumSpec :: Spec
thEnumSpec = describe "TH Enums" $ do
  prop "always round trips JSON instances" $ \ (x :: Fantastic) ->
    decode (encode x) `shouldBe` Just x

  prop "always encodes to something we expect" $ \ (x :: Fantastic) ->
    encode x `shouldSatisfy` flip elem ["\"mrFox\"", "\"beasts\"", "\"fantasia\""]
```

Now to test.

```bash
stack ghci
import Test.Hspec
hspec thEnumSpec
```