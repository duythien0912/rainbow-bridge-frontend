`@eth~near/core` – the Rainbow Bridge client library 🌈🌉
====================================================

Do you want to allow your users to send assets between [Ethereum] & [NEAR] over
the [Rainbow Bridge]?

Do you want to easily send assets between the two blockchains using your
command line?

Did you build a custom Rainbow Bridge [Connector] and now you want to figure
out how to build a client library for it, so other people can actually use it?

If you answered "Yes" to any of the above questions, this is the library for you!

  [Ethereum]: https://ethereum.org/
  [NEAR]: https://near.org/
  [Rainbow Bridge]: https://near.org/blog/eth-near-rainbow-bridge/
  [Connector]: https://github.com/near/rainbow-token-connector

Read on to find out how to:

- [Add it to your browser app](#add-it-to-your-browser-app)
- [Author a custom connector library](#author-a-custom-connector-library)


Add it to your browser app
==========================

Let's say you want to allow users to send ERC20 tokens from Ethereum to NEAR,
where they'll become NEP21 tokens.

Step 1: Add Dependencies
------------------------

You'll need to add two dependencies to your app:

    npm install --save @eth~near/core @eth~near/erc20~nep21

Alternatively, if using yarn:

    yarn add @eth~near/core @eth~near/erc20~nep21

Step 2: Initiate a transfer
---------------------------

Let's say you have a form.

```html
<form>
  <input id="erc20Address" />
  <input id="amount" />
  <input id="sender" />
  <input id="recipient" />
</form>
```

Here's the JavaScript you'll want to make this work:

```js
import { naturalErc20ToNep21 } from '@eth~near/erc20~nep21'

document.querySelector('form').onsubmit = e => {
  e.preventDefault()
  const { erc20Address, amount, sender, recipient } = e.target.elements
  naturalErc20ToNep21({
    erc20Address: erc20Address.value,
    amount: amount.value,
    sender: sender.value,
    recipient: recipient.value,
  })
}
```

What is `@eth~near/erc20~nep21`?

The Rainbow Bridge between Ethereum and NEAR has [many pieces][Rainbow Bridge].
One piece is **Connector** contracts. The connector code for converting ERC20
tokens in Ethereum to NEP21 tokens in NEAR lives at
[github.com/near/rainbow-token-connector][Connector].

The code for using a given connector from an app has its own library. The one
for the connector above is [`@eth~near/erc20~nep21`].

Anyone can make connector contracts, and anyone can make client libraries for
these contracts. If they follow the format of `@eth~near/erc20~nep21`, these
client libraries will work automatically with the core Rainbow Bridge transfer
library at `@eth~near/core`.

Generally, each connector client library, like `@eth~near/erc20~nep21`, will
export four main functions, which can be used to:

1. Go from a "natural" Ethereum token to a "bridged" NEAR equivalent
2. Go from a "bridged" NEAR token, meaning a token that started its life in
   Ethereum but which now lives in NEAR, back to Ethereum
3. Go from a natural NEAR token to a bridged Ethereum equivalent
4. Go from a bridged Ethereum token back to NEAR

For `@eth~near/erc20~nep21`, these main exports are:

1. `naturalErc20ToNep21` – example: go from DAI (a popular ERC20 token) to DAIⁿ
2. `bridgedNep21ToErc20` – example: convert DAIⁿ back to DAI
3. `naturalNep21ToErc20` – example: go from a natural NEAR token, such as BNNA
   Tokens in berryclub.io, to BNNAᵉ in Ethereum
4. `bridgedErc20ToNep21` – example: convert BNNAᵉ back to BNNA


Step 3: List in-progress transfers
----------------------------------

For the rest of the lifetime of the transfer you just initiated, you will use
exports from `@eth~near/core`, rather than the connector-specific library.

Let's say you want to list in-progress transfers in this `ol`:

```html
<ol id="transfers-go-here"></ol>
```

Here's code to render the list of transfers:

```js
import { get, onChange } from '@eth~near/core'

function renderTransfers () {
  const transfers = get({ filter: { status: 'in-progress' } })
  document.querySelector('#transfers-go-here').innerHTML =
    transfers.map(renderTransfer).join('')
}

onChange(renderTransfers)

renderTransfers()
```

If using React, you'd want something like:

```jsx
TODO
```

And here's what `renderTransfer` might look like, using vanilla JS (translation
to React is straightforward):

```js
import { act, decorate } from '@eth~near/core'

function renderTransfer (transfer) {
  // "decorate" transfer with realtime info & other data that would bloat localStorage
  transfer = decorate(transfer, { locale: 'en_US' })
  return `
    <li class="transfer" id="${transfer.id}">
      ${transfer.amount}
      ${transfer.sourceTokenName} from
      ${transfer.sender} to
      ${transfer.recipient}
      ${!transfer.callToAction ? '' : `
        <button class="act-on-transfer">
          ${transfer.callToAction}
        </button>
      `}
    </li>
  `
})

// Vanilla JS shenanigans: add a click handler to `body`, because transfers are
// rendered with JS and therefore unavailable for adding click handlers at
// initial page load.
// This will be easier if you use React or something 😄
document.querySelector('body').addEventListener('click', event => {
  const callToAction = event.target.closest('.act-on-transfer')
  if (callToAction) {
    const transferId = callToAction.closest('.transfer').id
    act(transferId)
  }
})
```

Here's some [docs about act][act], and [two][act2] [example][act3]
connector-specific behaviors. Here's some [docs about decorate][decorate], and
[two][decorate2] [example][decorate3] connector-specific behaviors. Here's the
attributes for [two][initiate-natural] [kinds][initiate-bridged] of raw
transfers, prior to being decorated.

  [act]: https://github.com/near/rainbow-bridge-frontend/blob/bfcd96178316f840845217371bebd253cc64abd1/src/js/transfers/index.js#L132-L140
  [act2]: https://github.com/near/rainbow-bridge-frontend/blob/bfcd96178316f840845217371bebd253cc64abd1/src/js/transfers/erc20%2Bnep21/natural-erc20-to-nep21/index.js#L62-L69
  [act3]: https://github.com/near/rainbow-bridge-frontend/blob/bfcd96178316f840845217371bebd253cc64abd1/src/js/transfers/erc20%2Bnep21/bridged-nep21-to-erc20/index.js#L67-L73
  [decorate]: https://github.com/near/rainbow-bridge-frontend/blob/bfcd96178316f840845217371bebd253cc64abd1/src/js/transfers/index.js#L46-L68
  [decorate2]: https://github.com/near/rainbow-bridge-frontend/blob/bfcd96178316f840845217371bebd253cc64abd1/src/js/transfers/erc20%2Bnep21/natural-erc20-to-nep21/index.js#L19-L59
  [decorate3]: https://github.com/near/rainbow-bridge-frontend/blob/bfcd96178316f840845217371bebd253cc64abd1/src/js/transfers/erc20%2Bnep21/bridged-nep21-to-erc20/index.js#L21-L64
  [initiate-natural]: https://github.com/near/rainbow-bridge-frontend/blob/bfcd96178316f840845217371bebd253cc64abd1/src/js/transfers/erc20%2Bnep21/natural-erc20-to-nep21/index.js#L97-L117
  [initiate-bridged]: https://github.com/near/rainbow-bridge-frontend/blob/bfcd96178316f840845217371bebd253cc64abd1/src/js/transfers/erc20%2Bnep21/bridged-nep21-to-erc20/index.js#L98-L121


Step 4: check & update status of in-progress transfers
------------------------------------------------------

Your app will need to prompt users to sign in with both Ethereum
([example][authEthereum]) and NEAR ([example][authNear]). After the
authorization process completes for both chains, you need this:

```js
import { checkStatusAll } from '@eth~near/core'

checkStatusAll({ loop: 15000 })
```

What's it do?

This library is designed to be non-blocking, which means a user can start
multiple transfers at once, and the library won't pause to wait for blocks to
be mined in Ethereum, finalized in NEAR, or synced between the two.

This means that with only the code from Steps 1-3, nothing else will happen. A
user will have sent an initial transaction to the Ethereum or NEAR blockchain,
but neither your app nor any other service will ever check to see if that
transaction completes successfully. Nor will any app or service prompt the user
to complete the next transaction in the process (any transfer requires multiple
steps & multiple on-chain transactions to complete).

`checkStatusAll` will loop as frequently as you tell it to. It will check to
see if transactions have been mined, synced, or finalized, and update transfers
in localStorage accordingly. When transfers are updated, the `onChange`
function in Step 3 will trigger a UI update.

  [authEthereum]: https://github.com/near/rainbow-bridge-frontend/blob/bfcd96178316f840845217371bebd253cc64abd1/src/js/authEthereum.js
  [authNear]: https://github.com/near/rainbow-bridge-frontend/blob/bfcd96178316f840845217371bebd253cc64abd1/src/js/authNear.js


Step 5: there is no step 5!
---------------------------

That's it! You successfully integrated cross-chain transfers into your app in
just four steps. 🌈🌉🎉

To make it more beautiful, check out [the API docs](#TODO🙃) and [example
code][example] (implemented in vanilla/no-framework JavaScript).

  [example]: https://github.com/near/rainbow-bridge-frontend/blob/bfcd96178316f840845217371bebd253cc64abd1/src/html/transfers.html#L339-L388


Author a custom connector library
=================================

1. Copy the code in the [`@eth~near/erc20~nep21`] library
2. Adjust for your needs

  [`@eth~near/erc20~nep21`]: https://github.com/near/rainbow-bridge-frontend/tree/526ed49248974e38b438d92c12ede1b6305eb869/src/js/transfers/erc20%2Bnep21