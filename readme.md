# DSPause

_enforce a delay on code execution to give affected parties time to react_

`ds-pause` allows authorized entities to make `plans`. A `plan` describes a single `delegatecall`
operation and a unix timestamp `era` before which it cannot be executed.

Once `era` has passed, a `plan` can be executed by anyone.

A `plan` can only be made if it's `era` is after `block.timestamp + delay`. The `delay` is
configurable upon construction.

A `plan` consists of:

- `usr`: the address to `delegatecall` into
- `fax`: the `calldata` to use
- `era`: the time before which the `plan` cannot be executed

## Auth

`ds-pause` uses a slightly modified form of the `ds-auth` scheme. Both `setOwner` and `setAuthority`
can only be called by the pause itself. This means that they can only be called by using `schedule` /
`execute` on the pause, and changes to auth are therefore also subject to a delay.

## Interface

**`constructor(uint delay)`**

- Initializes a new instance of the contract with a delay in ms

**`plan(address usr, bytes memory fax uint era) public auth`**

- Plan a call to address `usr` with `fax` calldata that cannot be executed until `block.timestamp >=
  era`
- Fails if `block.timestamp + delay > era`
- Returns all data needed to execute or cancel the scheduled call

**`drop(address usr, bytes memory fax, uint era) public auth`**

- Cancels a planned execution

**`exec(address usr, bytes memory fax, uint era) public returns (bytes memory response)`**

- `delegatecall` into `usr` with `fax` calldata
- Fails if the call has not been planned beforehand
- Fails if `era > block.timestamp`
- Returns the `delegatecall` output

## Tests

- [`pause.t.sol`](./pause.t.sol): unit tests
- [`integration.t.sol`](./integration.t.sol): usage examples / integation tests
