# Sign-In with Ethereum 

Elixir library to enable [Sign-In with Ethereum](https://login.xyz) message validation.

Full documentation found at <https://hexdocs.pm/siwe>.

This library provides functions for parsing and validating [SIWE](hhttps://eips.ethereum.org/EIPS/eip-4361) message strings and their corresponding signatures.

## Installation

The package can be installed by adding `siwe` to your list of dependencies in `mix.exs`:

```elixir
def deps do
  [
    {:siwe, "~> 0.3"}
  ]
end
```
## Example

To see how this works in `iex`, clone this repository and from the root run:
```bash
$ mix deps.get
```

Then create two files
`message.txt`:
```
login.xyz wants you to sign in with your Ethereum account:
0xfA151B5453CE69ABf60f0dbdE71F6C9C5868800E

Sign-In With Ethereum Example Statement

URI: https://login.xyz
Version: 1
Chain ID: 1
Nonce: ToTaLLyRanDOM
Issued At: 2021-12-17T00:38:39.834Z
```

`signature.txt`:
```
0x8d1327a1abbdf172875e5be41706c50fc3bede8af363b67aefbb543d6d082fb76a22057d7cb6d668ceba883f7d70ab7f1dc015b76b51d226af9d610fa20360ad1c
```
then run 
```
$ iex -S mix
```
Once in iex, you can then run the following to see the result:
```
iex> {:ok, msg} = File.read("./message.txt")
...
iex> {:ok, sig} = File.read("./signature.txt")
...
iex> Siwe.parse_if_valid(String.trim(msg), String.trim(sig))
{:ok, %{
  __struct__: Siwe,
  address: "0xfA151B5453CE69ABf60f0dbdE71F6C9C5868800E",
  chain_id: "1",
  domain: "login.xyz",
  expiration_time: nil,
  issued_at: "2021-12-17T00:38:39.834Z",
  nonce: "ToTaLLyRanDOM",
  not_before: nil,
  request_id: nil,
  resources: [],
  statement: "Sign-In With Ethereum Example Statement",
  uri: "https://login.xyz",
  version: "1"
}}
```

Any valid SIWE message and signature pair can be substituted.The functions described below can also be tested with `msg`, `sig`, or a value set to the result `Siwe.parse_if_valid`.

## API Overview
This library deals with three different types of input:

1) SIWE message strings.
2) Signatures of SIWE message strings.
3) A parsed SIWE message which is defined as:

```elixir
  defmodule Message do
    defstruct domain: "",
      address: "",
      statement: "",
      uri: "",
      version: "",
      chain_id: "",
      nonce: "",
      issued_at: "",
      expiration_time: nil, # or a string datetime
      not_before: nil, # or a string datetime
      request_id: nil, # or string
      resources: []
  end
```

The most basic functions are `parse` and `to_str` which translate a SIWE message string to a parsed SIWE message and back (respectively). To simplify using the variables from the above example:

```
iex> {:ok, parsed} = Siwe.parse(String.trim(msg))
...
iex> {:ok, str2} = Siwe.to_str(parsed) 
iex> str2 == String.trim(msg)
:true
```

Once parsed, the `Message` can be validated. 

- `validate_time` returns true if current time is after the `Message`'s `not_before` field (if it exists) and before the `Message`'s `expiration_time` field (if it exists). 

- `validate_sig` takes the `Message` and a corresponding `signature` and returns true if the `Message`'s `address` field would produce the `signature` if it had signed the `Message`'s string form.

- `validate` returns true only if both `validate_time` and `validate_sig` would.

```
iex> Siwe.validate_sig(parsed, String.trim(sig))
:true
iex> Siwe.validate_time(parsed)
:true
iex> Siwe.validate(parsed, String.trim(sig))
:true
```

`parse_if_valid` is an optimized helper function which takes a SIWE message string and a `signature` then returns a parsed `Message` only if the `signature` matches and the current time is after the `Message`'s `not_before` field (if it exists) and before the `Message`'s `expiration_time` field (if it exists). 

```
iex> Siwe.generate_nonce()
"EaLc76FkngQ"
```

Another helper, `generate_nonce` function is provided to create alphanumeric [a-z, A-Z, 1-9] nonces compliant with SIWE's spec, used like so:
This is useful for servers using SIWE for their own authentication systems.
## Disclaimer 

Our Elixir library for Sign-In with Ethereum has not yet undergone a formal security 
audit. We welcome continued feedback on the usability, architecture, and security 
of this implementation.

## See Also

- [Sign-In with Ethereum: TypeScript](https://github.com/spruceid/siwe)
- [Sign-In with Ethereum: Rust](https://github.com/spruceid/siwe-rs)
- [Example SIWE application: login.xyz](https://login.xyz)
- [EIP-4361 Specification](https://eips.ethereum.org/EIPS/eip-4361)
- [EIP-191 Specification](https://eips.ethereum.org/EIPS/eip-191)
