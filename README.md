## 1) User & Decision
Target user: Social Media content creators or brands wanting to maximize engagement (likes, comments, shares) on their posts.
Decision: Whether to post the content as it is, or make small adjustments such as to the timing of the post, photo content, or caption tone before publishing to improve engagement. 

## 2) Target & Horizon
Target: A numeric engagement level such likes, comments, or shares on a post.
Timeline: Engagement within the first 24 hours of posting. 
Type: Done by regression on weights to determine engagement score.

## 3) Features (No Leakage)
Post metadata: Caption sentiment analysis and length, posting time of day, hashtags
Photo analysis of content: Content, People, Photo Quality. 
Account history, analysis of engagement of past posts, posting frequency, comment analysis and what people want to see. 
Exclusions:
Prediction time leakage: Future engagement data such as likes, comments, or information attained after posting. For example, after the user presses a post, the API should not use its current engagement stats to factor in improvements to the post. 
Training Time Leakage: During training, we cannot use any data that occurred after a training post was published in order to predict it because then it may suffer leakage. When deciding a training set, we must make sure future posts never leak backwards in training. 

## 4) Baseline → Model Plan
Baseline: A rule-based model that returns the moving average of engagement (likes) for the user’s last 5 posts can be a suitable minimal benchmark that would be simple to implement and require little amounts of data. The assumption here is that the user's current post will be similar to their recent posts in engagement. This can be helpful as future models should exceed the performance of this baseline as they will have more access to data.

## 5) Metrics, SLA, and Cost:
Metrics: Since we are using regression, Mean Absolute Percent Error (MAPE) could be an appropriate way of measuring the degree of accuracy of the model in a human readable way. Since it reports things as percentages, it would be useful to see how well the model performs across different magnitudes (small content creators vs. larger content creators). However, for predictions with small engagement, predicting 10 likes vs. 1 like, then you can have crazy huge errors (900%). To counteract this, we will also track Mean Absolute Error (MAE) as a secondary metric, which is more stable for small number cases.
SLA: p95 latency < 500ms per prediction request. Cost envelope = free-tier hitting 100 requests/day and should automatically scale at around < $200/day at 50k requests/hour spikes. (More benchmarking and research is required to get a more accurate representation of costs and max costs)

## 6) Minimal Evaluation Plan
Train our model while avoiding leakages (done by splitting recent posts as our test set, while keeping historical posts as our training set). 
Use Mape as our primary metric to optimize for during training, and MAE as a potentially secondary metric. Then compare the trained model against our moving average baseline to see which performs better.
Measure the end-to-end request time from a client request to a response. Can be done by looking at the Networks tab on the browser developer console for some sample real world use case timings. 
Test normal use cases like 100 req/day which should sufficiently meet our p95 < 500ms. Do some spike testing for 50k req/hour for a short amount of time to see how the system reacts. Measuring the time taken for each request to calculate other p50/p95/p99. 
For costs, estimate reads/writes per request against the Database with user information. As well as invocation counts + execution time + memory allocation in GB-seconds for each request which can be used to calculate actual costs for running each lambda request. 
