# Swerve

Swerve is a type-level server and client library heavily inspired by Haskell's Servant library. 
A major, difference is the use of row types instead of introducing numerous combinators. 

## Installation

***This library is not yet published to pursuit.***  
You can install this package by adding it to your packages.dhall:

```dhall
let additions =
  { swerve =
      { dependencies =
          [ "console"
          , "effect"
          , "form-urlencoded"
          , "heterogeneous"
          , "http-media"
          , "http-types"
          , "media-types"
          , "psci-support"
          , "record-format"
          , "simple-json"
          , "typelevel-prelude"
          , "wai"
          , "warp"
          ]
      , repo =
          "https://github.com/Woody88/purescript-swerve.git"
      , version =
          "master"
      }
    , warp =
      { dependencies =
        [ "node-fs-aff"
        , "node-net"
        , "node-url"
        , "wai"
        ]
      , repo =
          "https://github.com/Woody88/purescript-warp.git"
      , version =
          "master"
      }
    , wai =
        { dependencies =
            [ "http-types"
            , "node-buffer"
            , "node-http"
            , "node-net"
            , "node-streams"
            , "node-url"
            ]
        , repo =
            "https://github.com/Woody88/purescript-wai.git"
        , version =
            "master"
        }
    , http-types =
        { dependencies =
            [ "console"
            , "effect"
            , "psci-support"
            , "tuples"
            , "unicode"
            , "uri"
            ]
        , repo =
            "https://github.com/Woody88/purescript-http-types.git"
        , version =
            "master"
        }
    , http-media =
        { dependencies =
            [ "console"
            , "effect"
            , "exceptions"
            , "foldable-traversable"
            , "maybe"
            , "newtype"
            , "numbers"
            , "ordered-collections"
            , "proxy"
            , "strings"
            , "stringutils"
            , "unicode"
            ]
        , repo =
            "https://github.com/Woody88/purescript-http-media.git"
        , version =
            "master"
        }
  }
```
```console
user@user:~$ spago install swerve
```

## Usage 

### Basic Example
```purescript 
import Prelude

import Control.Monad.Reader (asks)
import Data.Maybe (Maybe)
import Effect (Effect)
import Effect.Class.Console as Console
import Network.Wai (Application)
import Network.Warp.Run (runSettings)
import Network.Warp.Settings (defaultSettings)
import Swerve.API.Combinators (type (:<|>), (:<|>))
import Swerve.API.MediaType (JSON, PlainText)
import Swerve.API.Spec (Capture, Header, Header', Query, ReqBody, Resource)
import Swerve.API.Verb (Post, Get)
import Swerve.Server (swerve)
import Swerve.Server.Internal.Handler (Handler)
import Swerve.Server.Internal.Header (withHeader)
import Type.Proxy (Proxy(..))
import Type.Row (type (+))

type UserAPI = GetUser :<|> PostUser
type UserHandler = Handler GetUser String :<|> Handler PostUser (Header' { hello :: String } HelloWorld)

type HelloWorld = { hello :: String }

type GetUser = Get "/user"
    ( Resource String PlainText
    + ()
    )

type PostUser = Post "/user/:id?[maxAge]&[minAge]" 
    ( Capture { id :: Int }
    + Query { maxAge :: Maybe Int, minAge :: Maybe Int } 
    + Header { accept :: String }
    + ReqBody String PlainText
    + Resource (Header' { hello :: String } HelloWorld) JSON
    + ()
    )
 
getUser :: Handler GetUser String 
getUser = pure "User"

postUser :: Handler PostUser (Header' { hello :: String } HelloWorld) 
postUser = do 
    accept <- asks $ _.header.accept
    Console.log accept
    withHeader { hello: "world!" } { hello: "World!" }

api :: UserHandler
api =  getUser :<|> postUser

app :: Application
app = swerve (Proxy :: _ UserAPI) api

main :: Effect Unit
main = do 
    let beforeMainLoop = Console.log $ "Listening on port " <> show defaultSettings.port
    void $ runSettings defaultSettings { beforeMainLoop = beforeMainLoop } app
```         