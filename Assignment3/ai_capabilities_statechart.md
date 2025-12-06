## First Approach: Gemini 3 Pro (in Github Copilot Pro in agent mode)

I first decided to experiment with Gemini 3 Pro. I chose it because GitHub Copilot Pro is free for university students, so I don't need to create a new account for some site or enter billing information anywhere. Additionally, it's convenient to add files to the model's context and have it generate files directly in Visual Studio Code. I decided to try Gemini 3 Pro specifically because it's a relatively new model released by Google, and they claim it to be much better than 2.5 Pro (which from my personal prior experience was trash).

### Prompts Used

#### Initial Prompt
1. "Create a state chart for the system described in sys_description.md file. Generate the statechart as plantuml code. Make sure you use composite states and history (where relevant) at least once.

Note that the model should consist of at least at least 4 states, 7 events, guards, at least three types of actions/operations, composite states (at least once, where relevant), history pseudo-states (at least once, where relevant)."

*Note: "sys_description.md" file attached to the initial Gemini prompt contained our system description as Markdown text*

#### Follow-Up Prompts
2. "Fix the following error I got when compiling the code:
Error found in diagram ScooterStateChart
ERROR
6
Syntax Error?
Some diagram description contains errors"

3. "There are no operations defined in Paused and Charging states. only one transition as sequnece of actions defined, there should be more. Also one transition doesnt have event defined on it."

4. "It is currently not clear in the chart, what conditions lead the scooter from charging state to outofservice state. Improve that."

5. "This is better now, but technicianpickup event from available to outofservice state is also not understandable. Improve it as well."

6. "I think the transitions, which have event technicianpickup, should instead use the text given as guard for the event text instead (and get rid of technicianPickup entirely)."

### Assessment

The model created a syntax error for the first prompt, which was problematic. However, it fixed the error after one follow-up prompt where I provided the PlantUML error message. The first diagram that compiled didn't look too bad, but some states missed operations, and some transitions were not clear. Overall, it was perhaps too simplistic compared to our diagram.

#### Successes
- Fixed syntax errors quickly when provided with specific error messages
- Generated a compilable PlantUML diagram on the second attempt
- Created a reasonable initial structure for the state chart
- Followed the basic requirements (composite states, history pseudo-states, multiple events, guards, actions)
- History pseudostate placement was quite reasonable and appearance was correct

#### Shortcomings
- Initial output contained syntax errors
- Some states missed operation definitions
- Transitions were unclear and lacked descriptive guards or conditions
- Diagram was too simplistic and could be improved with more detail
- Further refinement would require careful review and multiple iterations, which would be time-consuming to ensure new states and transitions fit properly with all necessary elements (operations, events, guards) defined
- Not suitable for improving existing statechart due to oversimplification; would have been more useful if applied during initial model creation rather than as a refinement tool

#### Conclusion

Good for generating an initial idea, but the output requires quite a bit of refinement to make it production-ready. The best approach to use this AI's generated model would be to create a new chat, provide the generated code as context along with the original system description, and explicitly request something similar but more detailed with clearly understandable transitions and all necessary elements (events, guards, operations) fully defined. For existing statechart, this approach would not be practical due to the oversimplification — it's better suited for starting fresh.

## Second Approach: Claude Sonnet 4.5 (in Github Copilot Pro in agent mode)

As a second approach, I used Claude Sonnet 4.5. I chose it because GitHub Copilot Pro is free for university students, so I don't need to create a new account for some site or enter billing information anywhere. Additionally, it's convenient to add files to the model's context and have it generate files directly in Visual Studio Code. I decided to compare Gemini 3 Pro to Claude Sonnet 4.5 because it's a baseline good model for coding (based on my prior experience, it was orders of magnitude better than Gemini 2.5 Pro).

### Prompts Used

#### Initial Prompt

1. "Create a state chart for the system described in sys_description.md file. Generate the statechart as plantuml code. Make sure you use composite states and history (where relevant) at least once.

Note that the model should consist of at least at least 4 states, 7 events, guards, at least three types of actions/operations, composite states (at least once, where relevant), history pseudo-states (at least once, where relevant)."

*Note: "sys_description.md" file attached to the initial Gemini prompt contained our system description as Markdown text*

#### Follow-Up Prompts

2. "This is too complex currently. Tone the complexity down (while still fulfilling the requirements). Also some of the states are missing operations. Also remove the comments inside the statechart: it should be understandable without comments."

3. "You only need one composite state. Almost all of your current composite states contain only one state, which is pointless."

4. "Got syntax error when compiling plantuml:
Error found in diagram Scooter_Sharing_System_Statechart
ERROR
7
Syntax Error?
Some diagram description contains errors"

5. "It's weird that scooter can go straight from available to decommissioned final state. It should first go to technician, if technician tells its unrepairable, then go to final state instead"

### Assessment

The model managed to generate the statechart without syntax errors after the first prompt, which was a good start. However, it made the diagram way too complex and used many pointless composite states that contained only one state inside. It also added comments to the statechart, but the diagram should be understandable without them. The history state was also in the wrong place, and it would have taken more time to get it fixed because I would need to discuss and explain where and why it can and should be used (it seems Claude doesn't understand the history pseudostate as well as Gemini 3 Pro).

#### Successes

- Generated syntactically correct PlantUML code on the first attempt
- States and transitions had properly defined operations and events
- Helped improve the existing statechart model by providing ideas (e.g., decommissioned scooter final state)
- Better overall structure and detail compared to Gemini 3 Pro
- More suitable for creating initial statecharts

#### Shortcomings

- Initial output was overly complex with unnecessary composite states
- Many composite states contained only one state inside, which is pointless
- Added comments inside the statechart when it should be self-explanatory
- History pseudostate was placed incorrectly
- Would require additional effort to explain and fix the history state placement
- Understanding of history pseudostates appears weaker than Gemini 3 Pro

#### Conclusion

Overall, Claude Sonnet 4.5 performed better than Gemini 3 Pro, particularly in defining operations and events for states and transitions. A significant advantage is that it helped improve our existing statechart model by providing useful ideas like the decommissioned scooter final state. However, it still needs refinement, especially regarding the history state placement. Despite these issues, I would say it's better for creating an initial statechart compared to Gemini 3 Pro.

---

## Overall Conclusion

Both models were good at generating initial statecharts that could be built upon. Claude Sonnet 4.5 provided a useful idea for improving our existing statechart (the decommissioned scooter final state), while Gemini 3 Pro did not help improve the current model due to its oversimplification. Overall, Claude Sonnet 4.5 still seems to be the better choice, although Gemini 3 Pro has caught up to the baseline Sonnet 4.5 model somewhat, offering comparable quality and even handling the history pseudostate better than Sonnet 4.5.
