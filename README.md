# Is The Uber Worth It?
## A Notre Dame Night Out Predictor 🍺

🍺 **[→ Try the live app](https://ccarls22-maker.github.io/Is-The-Uber-Worth-It-/)** 🍺

.

# For More information about the app and model behind it:
## Is it Worth the Uber? An AI Nightlife Predictor
.
# 1. Problem Solving & Underlying Motivation
##What business problem are you trying to solve?
This project was created to address the uncertainty and inefficiency college students face when
deciding whether to go out to the local bars or stay in on any given night. Specifically, we aimed
to solve the problem of deciding if a venue is dead or busy. This is one of the biggest factors that
impacts whether a night out is well spent or a waste of time. Our plan is to create a data pipeline
based on Google ratings, weather, events in the area, and other factors to help users make
informed decisions before leaving their homes.
.
## Why do you need AI to solve this problem?
Our approach is one of aggregation: we are combining data from multiple sources into a single
central algorithm to predict venue busyness. This is done through machine learning utilizing
XGBoost and linear regression models to identify complex, non-linear patterns that traditional
manual tracking cannot reveal. It also saves the user a great deal of time, as they can go to a
central location rather than searching for it separately.
.
## Why should we care about this application of AI?
The application of AI in this business problem matters because it maximizes the value of limited
social time while providing both strategic and utility use for both consumers, in this case, college
students and businesses around the South Bend area. For students, going out time can be limited
because of previous and school commitments, which makes it important to enjoy the time you
have away from studies. By centralizing the data from multiple sources, the system provides a
great deal of convenience to anyone who uses it. There is also a big opportunity to scale the
business, utilizing our AI algorithm and expanding into other areas such as apartments.com,
which would significantly increase the opportunity for a high-value acquisition.
.
.
# 2. Solving the Problem using AI
## Describe the data. How did you get it and what does it show?
We initially explored Kaggle to find our data, but quickly found out that what we were looking
for did not exist in a centralized dataset. This led our group to compile our data from different
sources, including Google Popular Times, weather data, event data, and other sources. Using
SerpAPI, we were able to scrape the data from Google with a script instead of having to do it
manually, and pulled the data from nine bars at the beginning. For each bar, we collected the
Popular Times scores, number of reviews, whether there is a cover, and distances from the Notre
Dame campus. This approach allowed us to create a “busyness score” with which we aim to
indicate whether a location is worth going to or should be avoided, depending on the level of
activity.
.
## Describe the AI that you're using. How does it work?
The model we built is a two-layer predictive system for bar busyness in South Bend. The first
layer is an XGBoost regression model that predicts baseline busyness from venue characteristics
and time features. The second layer is a rule-based adjustment layer that modifies Layer 1's
output using real-world context, specifically weather conditions, the Notre Dame academic
calendar, and the football schedule. Together, these layers produce an output from 0 to 100
representing a busyness score per venue per hour, which we then translate into categorical vibe
labels (Dead, Chill, Moderate, Lively, and PACKED) for the eventual end user.

We chose XGBoost because this tree-based ensemble model captures complex, non-linear
patterns without overfitting, and it consistently performs well on structured tabular data.
Throughout training, XGBoost iteratively learns from its own mistakes by building each new tree
to correct the errors of the previous one, which lets it surface unique patterns in the data. Given
our relatively small dataset, we opted against a neural network in favor of XGBoost, both
because it was a better fit for our data size and because we wanted to optimize for compute. If we
could avoid investing in heavy-duty GPU compute power without sacrificing performance, we
thought that was the smarter way to scope the project. We ultimately trained our model using 200
trees, a maximum depth of 6, and a learning rate of 0.1.

Layer 2 works by applying multiplicative modifiers, or coefficients, to the Layer 1 output. Our
weather modifier was derived from published foot-traffic research that quantified how snow
reduces traffic by roughly 40 percent, freezing temperatures by about 25 percent, and pleasant
weather boosts traffic by around 10 percent. Using these coefficients, we were able to scale
Layer 1's predictions into Layer 2's contextualized output. Similarly, our event modifiers came
from research on the ND academic calendar and home football schedule, which pointed to spring
break cutting traffic by about 60 percent and home football game weekends boosting busyness
by roughly 35 percent. As mentioned previously, we relied on this multiplier method because the
Google Popular Times API provides historical averages rather than daily observations, which
means we cannot empirically train on specific weather effects from the data itself.
For validation, we used a standard 80/20 train-test split, which produced a Mean Absolute Error
(MAE) of 3.67 and an R-squared value of 0.948. In addition, we ran a leave-one-venue-out
(LOVO) cross validation, where we trained on 7 bars and tested on the 8th bar that the model
had never seen, which yielded an MAE of 16.24. LOVO is by far the more rigorous test, and the
fact that the model still generalized reasonably well proves that it is learning from venue
characteristics rather than memorizing patterns.

For interpretability, we used SHAP values to surface the most important features in our data. The
top drivers turned out to be hour_sin (time of day), is_going_out_night (Friday/Saturday
late-night flag), number of reviews, distance from the ND campus, and raw hour. We also
included a cover charge feature that flagged bars with weekend entrance covers, and those
features ranked 9th and 14th in importance respectively. While they were not in the top tier, they
did provide meaningful secondary signal.
.
.
# 3. Moving Beyond the Model
## Describe the limits to your approach. What would completely change the model's capabilities/capacity to handle the problem?
In terms of limitations, our core data presents some considerable concerns. The biggest is that
Google Popular Times only provides historical averages rather than daily observations, which
means we cannot empirically train on weather or event effects. For example, a bar would have
the same busyness score for Friday at 11pm regardless of whether it was 70 degrees in June or 10
degrees in January. As a result, our Layer 2 coefficients come from published research rather
than being learned from our specific domain of bars in South Bend.

In addition, the dataset itself is quite small. With only 8 bars in the Notre Dame area, our total
training set was about 900 rows, while XGBoost can handle millions, so data sparsity was a real
constraint. Beyond size, the Google Busyness scores themselves skew toward Android users
since they make up the primary location tracking base, and venues with low review counts often
receive little to no Popular Times data. Olf's Blarney Stone is a clear example of this, since we
had to drop it from the model with only 64 reviews. Our cover charge feature also has a similar
limitation, as the cover policies were generalized and estimated rather than verified through
direct outreach to the bars or research into specific events.
Finally, the model shows some signs of overfitting. While it performs strongly on the standard
train-test split, it struggles considerably with the LOVO test (MAE of 16.24), and bars with
unique business models such as Finnies, which is only open Thursday through Saturday, posted a
LOVO MAE of nearly 29.
.
## What are some broad legal or ethical concerns?
In terms of legal and ethical concerns, one possibility we discussed is that bars themselves could
end up using this tool to their advantage. Specifically, bars could identify peak times and adjust
their cover charges accordingly, which would put customers in a disadvantageous position.
Similar to surge pricing on Uber or SeatGeek, this kind of dynamic pricing applied to nightlife
would mean students and other customers consistently get the short end of the stick.
We also recognized that with the right data, this model could begin detecting demographic
patterns within bar-goers. While that information could be useful for students who want to go
where their peers are, it could just as easily be used to discriminate against groups of people who
are not Notre Dame student-adjacent.
.
# 4. Making AI a Product, Service, or Business
## Does your model/pipeline fit best as a product, service, or potentially a completely separate business?
The model was originally designed as a standalone product for students, and that remains its
most natural application. Since the model is computationally lightweight, results could be easily
integrated into a website or app, making it ready for on-the-fly night-out decisions. Building on
that foundation, the model could also be offered to local bar owners and staff to support dynamic
pricing decisions on covers. Through personal experience, we recognize that bars could often
better control the nightly experience by adjusting cover charges to ensure stronger long-term
perceptions. Beyond its immediate use case, the model's core logic lends itself well as a
supplementary feature for existing services ideally operating like a "walkability" score on
Apartments.com, giving renters who are moving to a new area meaningful insight into the
culture of a neighborhood.
.
## How do you know if it is successful/works?
Ultimately, real-world success would mean students can go out and have the experience they
expected. In terms of the current model, it achieves a strong accuracy of 81%. Given how
tailored the model is to the South Bend region, and more specifically to bars that cater uniquely
to students, we would measure broader success by testing whether it generalizes beyond its
current scope. Right now, Google Reviews serves as a placeholder for popularity, and alongside
"night out" activity, it remains the biggest driver of model performance. However, Google
Reviews are a lagging indicator, they reflect past experiences rather than what's happening on a
given night. Looking ahead, incorporating live foot traffic data would allow the model to move
beyond proxies entirely, producing more accurate, real-time predictions and opening the door to
wider applications beyond South Bend.
.
## What is your target customer, client, or use case?
The primary target customers are students. The application was built with them in mind, drawing
on innate knowledge of local bars and the broader student experience. That said, the model can
be readily applied to local bar staff and owners looking to gauge covers, plan promotions, or
pursue goals tied to public perception. For example, if Finnie's (Newfs) wants to maintain its
reputation as the town's primary "clubbing" destination, it could adjust cover prices upward
during predicted "Lively" periods to account for heightened demand. On the flip side, bars losing
ground to competitors could use these insights to strategically reposition themselves. If CJ's sees
that Newfs is predicted to be "lively" on a given night, they could proactively drop their cover to
attract spillover from those unwilling to wait in Newfs' line.
Can you identify a potential company that would acquire you or immediately adopt your
services?

As mentioned, this service has potential well beyond student and bar use cases. Companies like
Apartments.com and Apartment List, which aim to curate the renter search experience, could
adopt a broader version of the model as a "nightlife vibrancy" score for a given area. This would
require adjusting the model to fit a wider context. For instance, variables like "football weekend"
could be swapped out for inputs derived from the Haversine formula, measuring an apartment's
distance to local bars and major event venues. The model could then incorporate metrics based
on average local events, such as concerts and sporting events, while also capturing bar density
and peak "lively" periods in a given area. This could be further customized for prospective
renters by establishing busy windows, such as flagging when bars in an area cater to an older
crowd and tend to peak earlier in the evening.
