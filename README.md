Tasty Travis: Fancy Travis CI output for tasty tests
====================================================
[![BSD3](https://img.shields.io/badge/License-BSD-blue.svg)](https://en.wikipedia.org/wiki/BSD_License)
[![Hackage](https://img.shields.io/hackage/v/tasty-travis.svg)](https://hackage.haskell.org/package/tasty-travis)
[![hackage-ci](https://matrix.hackage.haskell.org/api/v2/packages/tasty-travis/badge)](https://matrix.hackage.haskell.org/#/package/tasty-travis)
[![Build Status](https://travis-ci.org/merijn/tasty-travis.svg)](https://travis-ci.org/merijn/tasty-travis)

**Tasty Travis** provides fancy
[Tasty](https://hackage.haskell.org/package/tasty) test output on
[Travis CI](https://travis-ci.org/).

It allows you get coloured test output, folding and collapsing groups of tests,
and hiding the output of successful tests.

Example
-------

Here's what an example `test.hs` might look:

```haskell
import Data.List
import Data.Ord
import Data.Tagged (Tagged)
import Data.Typeable (Typeable)
import Options.Applicative (switch, long, help)

import Test.Tasty
import Test.Tasty.Options
import Test.Tasty.Travis (travisTestReporter, defaultConfig)
import Test.Tasty.HUnit

newtype EnableTravis = EnableTravis Bool
  deriving (Eq, Ord, Typeable)

instance IsOption EnableTravis where
  defaultValue = EnableTravis False
  parseValue = fmap EnableTravis . safeRead
  optionName = return "enable-travis"
  optionHelp = return "Run Travis tests."
  optionCLParser =
    fmap EnableTravis $
    switch
      (  long (untag (optionName :: Tagged EnableTravis String))
      <> help (untag (optionHelp :: Tagged EnableTravis String))
      )

main = travisTestReporter cfg [] tests
  where
    cfg = defaultConfig { travisTestOptions = setOption (EnableTravis True) }

tests :: TestTree
tests = testGroup "Unit tests"
  [ testCase "List comparison (different length)" $
      [1, 2, 3] `compare` [1,2] @?= GT

  -- the following test does not hold
  , testCase "List comparison (same length)" $
      [1, 2, 3] `compare` [1,2,2] @?= LT
  , askOption $ \(EnableTravis enable) ->
    if enable then travisTests else testGroup "" []
  ]

travisTests :: TestTree
travisTests = testGroup "Travis" [ {- Travis tests here -} ]
```
