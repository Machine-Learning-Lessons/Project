# Project

## Question: 
<i>I'm a bit confused? Is the option to just continue doing HWs still on the table? If that is an option, that would be the intelligent choice for me. Full disclosure - Idea 1 would probably end up using a lot of AI help and some vibe coding. </i>

## Idea 1 - Impact of qualitative coding values on outcome variables
In psych, when we have qualitative data (like images or open ended questions) we have human raters provide codes for them so that stats analyses can be run on them. Usually this process follows a codebook - i.e.: "If the answer mentions cats, code 5- if it mentions dogs code 6". Additionally, if we're being responsible, we use stats formulas like Cohen's Kappa or Cronbach's alpha to measure agreement between raters as a measure of reliability. Regretably, sometimes these tools aren't even used and (if the variable is continuous)  the researchers simply take the average of the two raters rating. That said, most studies use 1 or 2 raters/coders max and the threshold for 'agreement' is actually quite low and very dependent on the number of questions being asked. 

### Background
For another previous class project (Visualizing Uncertainty, Sp '24), my group and I designed a tool -tuned to one particular data set- that could show how an outcome variable might change based on differences in a coding from a single additional coder for a continuous variable. For that data set, it turned out that only massive disagreement would change the outcome variable (odds ratio). (The data set was 'skin color of soccer players' vs 'number of red cards received', the odds ratio did increase or decrease depending on judgements of how light or dark a player's skin was, but the outcome usually remained a stable increased likelihood of receiving red cards the darker one's skin was). 

### For this course
Following this, in wishing to continue this work, my goal   was to then see if this could be made generalizable across multiple data sets. Ideally, the tool would take in preliminary ratings, the question or stimuli, and the codebook, and return a range showing how sensitive the results are to likely variations in future additional ratings. 

The way I could see this working with machine learning is to take public data sets which have made their coding public, and use an LLM or a script to calculate per item disagreement. We could then use a ML model using Bayesian heirarical regression to determine if there are patterns between codebook instructions, the wording of questions, and rater disagreement. Using this information the hope would be to be able to create a way of identifying weak coding schemas, and to simulate per question variance to a degree that the impact on an outcome variable could be measured. 



## Idea 2 - Increase the intelligence of a rudimentary robot vac
The other completely unrelated idea was to continue work on a project I started over the summer and shelved once classes started. This was an attempt to make a very dumb robot vacuum (in my defense, it was a gift) slightly more intelligent. I have already acquired a number of sensors, accelerometers, cameras, and esp32s - and the plan was to use the vacuum's 'Auto' mode to create a heat map of the boundaries of the apartment, then have a script occasionally drive it to areas of the apartment that need work - or to unstuck itself. ML would fit for trying to classify sensor readings while the robot is on auto, and/or classifying objects on camera as known stable obstacles or temporary hazards (cats), and/or learning a standard error by which location estimation is off by. 
