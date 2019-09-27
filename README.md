# Data Weave RFC

Substantial change proposals for the DataWeave language must be first
written up as an RFC and before they can be accepted.  The Request for
Comment (RFC) process is intended to provide a peer review and voting
process to ensure change proposals are vetted and accepted by the
community.

## The Process
[process]: #process

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

Inspiration for this RFC process is taking from the processes used by Posy, Whiley, and Rust