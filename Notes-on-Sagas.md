---
Description: Notes on Sagas
Author: Vassilis Vatikiotis
Date: 7 December 2017
Notebook: Work
Tags: Distributed Systems
Source: https://www.youtube.com/watch?v=xDuwrtwYHu8
---

# Notes on Sagas from Applying the Saga Pattern

**Sagas are long lived / Distributed transactions.**

**Sagas trade atomicity for availability.**

(Atomicity requires each transaction to be all or nothing: If a part o
transaction fails then the entire transaction fails and the system state is left
untouched).

**Sagas are a failure management pattern.**

---

Some points:

* Consider thinking in terms of "requests" wherever you see "transactions".
  Transactions imply ACIDity and we dont necessarily have it.
* A Saga is a collection os sub-transactions.
* If you execute a transaction (e.g. email) you cannot undo it (e.g. unsend the
  email). What you can do is issue another transaction that will _semantically_
  undo it (e.g. sent another email
  _cancelling/rectifying/fill-in-domain-specific-action_ ). A compensating
  transaction.
* A Saga either is successful (all sub-transactions are successful) or a Saga
  has failed (some transactions have failed and the corresponding compensating
  transactions have completed). Semantically, either way, the system is in a
  consistent state.
* A compensating transaction _must_ be played indefinitely until it succeeds.

- Backwards recovery (aka start rollback transactions, the compensating
  transactions bit I noted earlier).

### Unrelated but noteworthy, sometimes sprinkled with whoahh

* Google Spanner: Google's scalable, multi-version, globally distributed,
  synchronously replicated database. Supports externally consistent distributed
  transactions. _Installed atomic clocks in each datacenter and datacenters are
  connected with fiber_.
  * It achieves total ordering within the Google datacenter universe with max
    delays of 7ms.
  * Custom hardware. Very expensive.
  * Synchronization is not solved, it just narrows down the time window of
    possible transaction _conflicts_.
