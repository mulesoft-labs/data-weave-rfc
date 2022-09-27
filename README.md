# Data Weave Language

The DataWeave Language is in the process of being open-sourced. You can read our announcement [here](https://blogs.mulesoft.com/news/dataweave/). Our journey has just begun and it will take some time for the code to be available. In the meantime, we want to start engaging with our community to understand how DataWeave could be used and integrated. 

If you are interested on leveraging DataWeave:
 1. Join our community [Slack](https://join.slack.com/t/dataweavelanguage/shared_invite/zt-1f3xmq8n6-MVoUj7dDamxyu_Zyf62ERQ)
 2. Join the `#opensource` channel

For more news and all things DataWeave, visit our [site](https://dataweave.mulesoft.com/) 

# Data Weave RFC

Substantial change proposals for the DataWeave language must be first
written up as an RFC before they can be accepted.  The Request for
Comment (RFC) process is intended to provide a peer review and voting
process to ensure change proposals are vetted and accepted by the
community. We acknowledge that proposals might require extensive discussions
even before being written up, and encourage the use of [issues](https://github.com/mulesoft-labs/data-weave-rfc/issues) as a way to
engage in such discussions.

## The Process
[process]: #process

If you'd like the subject to be discussed before jumping into a formal RFC, you can create an [issue](https://github.com/mulesoft-labs/data-weave-rfc/issues/new) explaining the idea or the problem you'd like to solve. A corresponding [proposal](https://github.com/mulesoft-labs/data-weave-rfc/tree/master/proposals) can be merged simultaneously to later expedite the RFC process. Once you are ready to move forward, the proper RFC can be created.

To get a substaintial change accepted into the DataWeave language, you must first get the corresponding RFC merged into this repository as a markdown file. At this point the RFC is "active" and may be implemented and eventually included in DataWeave. 

The process is as follows:

* **Fork** this RFC Repository
* **Copy** `0000-template.md` to `text/0000-my-feature.md`. Here, `my-feature` should be a descriptive name for the feature. Don't assign an RFC number yet since this may need to be changed later anyway.

You can use the following command to copy the file:

```
# From the root of the project
cp 0000-template.md text/0000-my-feature.md
```

* **Complete** the RFC by providing as many details of the proposal as possible. The RFC must at a minimum: 1) motivate the need for the change; 2) clearly explain the technical details of the change (i.e. what parts of the language will be changed and in what way); 3) highlight any potential impacts of the change (esp. if the change is not-backwards compatible); 4) consider alternative solutions to the problem.
* **Submit** a Pull Request (PR) against this repository. This will instigate the review period during which the RFC will receive feedback and suggestions from the community.
* **Update** the RFC during the review period to address concerns or issues raised during review. Such changes should be made as new commits to the Pull Request, along with comments clarifying the changes. RFCs are rarely accepted without changes.
* **Acceptance.** At some point, once the discussion has slowed, the motion will be made for a decision to be made regarding the RFC (often referred to as the Final Comment Period (FCP)). After a short period (e.g. one week), the RFC will then either be accepted (and merged) or rejected (and closed). In some cases, the RFC maybe be postponed and kept open.

## Acknowledgements
[acknowledgements]: #acknowledgements

Inspiration for this RFC process is taken from the processes used by [Pony](https://github.com/ponylang/rfcs), [Whiley](https://github.com/Whiley/RFCs), and [Rust](https://github.com/rust-lang/rfcs#reviewing-rfcs).
