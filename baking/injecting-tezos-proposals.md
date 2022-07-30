---
description: >-
  Always inject responsibly.  If your proposals wins, you're going to have to
  live with the outcome.  Try your best to test and participate in the rest.
---

# Injecting Tezos proposals

## Introduction

The Tezos protocol gets modified mostly by the developers working at [https://gitlab.com/tezos/tezos](https://gitlab.com/tezos/tezos). It's a collaboration of several developer organizations, which you can research by navigating through the contributor's names.

{% hint style="info" %}
It's acceptable for a developer or a developer community to work in secret on a fork of the repository above to create a well constructed proposal which passes all existing tests as well as its own set of tests, intended to best discover any flaws.
{% endhint %}

{% hint style="info" %}
**Without the appropriate testing, no proposal should be injected.  While there is testing built into the proposal process itself, this does not absolve a developer from the developer's due care (find fault using existing tools) and due diligence (find fault using own tools).**
{% endhint %}

## Injection Scenarios

### Scenario 0: Core dev consensus

This is simple.  Every once in a while the developers work through consensus and post about their proposals, primarily in [https://research-development.nomadic-labs.com/blog.html](https://research-development.nomadic-labs.com/blog.html)

Once this happens, all you have to do is git checkout the repo where the proposal is finalized, sync  a tezos node and inject.

### Scenario 1: Core dev non-consensus

In this scenario you will be contacted by a core dev who has composed a proposal outside of the main Gitlab repo (for whatever reason) and you are asked to inject the proposal because of your agreement with the parameters being changed.

### Senario 2: Non-core dev non-consensus

In this scenario you will be contacted by a non-core dev group that has ample resources to work on the Tezos protocol competently.  They will make sure all requisite testing is successful and they will write their own testing, in an honest effort to discover any issues within their code changes, no matter how small.
