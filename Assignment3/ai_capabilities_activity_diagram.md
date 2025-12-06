## First Approach: Gemini 3 Pro

I first decided to experiment with Gemini 3 Pro. I chose it because it's a relatively new model released by Google and has been very well received, doing much better than its predecessor according to benchmarks and general user sentiment I've noticed. This should give the AI tool in question the best chance at matching or even improving our own diagram.

### Prompts Used

#### Initial Prompt
1. "Create an activity diagram for the following system: <<Entered our system description from task 1 here>>. Generate the activity diagram as plantuml code. 

#### Follow-Up Prompts
2. "This version is a bit too technical, you are modelling activities so describe those activities instead of things like "execute method x""

3. "Consolidate all the system related swimlanes into one named "System" for clarity, currently there are multiple swimlanes with only two or three elements."

### Assessment

The model did a fantastic job and reached an activity diagram very similar to our own by the end. Separating or consolidationg the swimlanes is a design decision more than anything else, so the only thing needing correction was telling Gemini to use natural language to describe actions.

#### Successes
- Generated a compilable and readable PlantUML diagram on the first attempt
- Created a reasonable initial structure for the activity diagram
- Perfectly handled consolidating swimlanes

#### Shortcomings
- Initial output was machine-like, referencing specific functions and classes rather than describing actual actions

#### Conclusion

Gemini 3 Pro understood the task immediately and made one small mistake before creating an activity diagram even a bit better than our own. I was able to improve our diagram with more specifics, for example including the condition for updating the scooter status to one option or another before the end.

## Second Approach: Claude Sonnet 4.5 (in Github Copilot Pro in agent mode)

As a second approach, I used Claude Sonnet 4.5. I decided to compare Gemini 3 Pro to Claude Sonnet 4.5 because it's a baseline good model for coding (based on my prior experience, it was orders of magnitude better than the earlier Gemini 2.5 Pro).

### Prompts Used

#### Initial Prompt

1. "Create an activity diagram for the following system: <<Entered our system description from task 1 here>>. Generate the activity diagram as plantuml code.

#### Follow-Up Prompts

2. "This version is a too technical, you are modelling activities so describe those activities instead of just writing method calls"

3. "This is much better but excessive, make no assumptions and only include steps from the description."

4. "Consolidate all the system related swimlanes into one named "System" for clarity, currently there are multiple swimlanes with only two or three elements."

### Assessment

This model did a fine job, though it did initially add it's own assumed extra actions to the diagram which needed to be removed. 

#### Successes

- Generated syntactically correct PlantUML code on the first attempt
- Diagram included all the actions laid out in the given description
- Responses to follow-up prompts were as requested and without errors

#### Shortcomings

- Initial output was overly complex, laying out specific function calls and all the steps between controller steps rather than focusing on the actual actions performed by those functions
- The model assumed extra steps in the overall process (not mentioned in the description) and included them in the diagram despite no request to do so

#### Conclusion

Overall, Claude Sonnet 4.5 model's attempt went about the same as it did for Gemini 3 Pro, with an initial decent but overly technical (and in Sonnet's case bloated with extra functionalities) version, which was quickly improved upon to reach a similar and slightly improved diagram to our existing one. There wasn't really anything to add to our diagram that wasn't already present in Gemini 3 Pro's version.

---

## Overall Conclusion

Both models were good at generating activity diagrams similar to our own with minimal further instruction. Gemini 3 Pro provided a useful idea for improving our existing activity diagram (specifying scooter status change and conditions for it), while Claude Sonnet 4.5 did the same and didn't change anything further. Overall, both models were great option for activity diagram generation (with PlantUML), with Gemini 3 Pro barely coming out ahead needing slightly less follow-up to reach the desired result.
