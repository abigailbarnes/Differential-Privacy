# Differential-Privacy
This week, there are three tasks, two of which are closely related synthesis tasks and one of which is a short exploration task. As usual, you'll be turning in both code and a PDF write-up.

Task 1: Synthesis: Randomized Response (26 points)
As we discussed in class, the origin of differential privacy is partially in randomized response. In a randomized response protocol, the person taking a survey is asked (metaphorically) to privately flip a coin. If the coin lands on heads, then the person should answer the yes or no question truthfully. If the coin lands on tails, then the person should privately flip the coin again and respond yes if heads and no if tails.

Write-up Question 1 (14 points): Write a program to simulate this process with a population of 1000 people, running the simulation 100 times each. Do this four times, where the prevalence of people for whom the answer to the survey question is yes is respectively 50%, 25%, 10%, and 1%. For each of these four percentages, plot a histogram of proportion of "yes" responses across the 100 tiles when using randomized response. Include the histogram in your write-up. Also write 1 or 2 sentences describing what you notice about the distribution for each and how these distributions relate to the true prevalence of "yes" answers in the population.

Write-up Question 2 (7 points): 2. In the randomized response experiment you just ran, the coin flip was parameterized at 1/2 probability of answering truthfully. Try biasing the coin so that the probability deciding to not answer truthfully is 1/8, and 1/4, and 3/4. Keep the coin flipping process from before (the one with 1/2 probability) for selecting the answer when you are not answering truthfully. Run the simulations above again for each of these three situations, producing a total of 12 additional histograms. Include these in your write-up and comment in a sentence or two about what you observe.

Write-up Question 3 (5 points): The probability of a truthful answer in randomized response is inversely proportional to the amount of "deniability" one has about the answer that one provides to the questioner. Is there a threshold probability at which you would feel comfortable providing information to:

The government?
A researcher?
A publically accessible database?
Social media?
Write at most two sentences for each, explaining your answer.

Task 2: Synthesis: Differential Privacy (50 points)
Differential privacy is a definition, rather than a technique. The randomized response you saw in Task 1 is an example of a technique that satisfies (ln 3, 0) differential privacy. Because we discussed differential privacy at a fairly high level in class, you may want to refer to this tutorial Links to an external site.and/or this textbook Links to an external site.(especially definition 2.4, which further formalizes differential privacy, and Section 3.3, which discusses the Laplace mechanism).

As discussed in class, a common method for achieving differential privacy on numerical data is the Laplace mechanism. While randomized response is distributed (local differential privacy), in that the data aggregator receives only noisy data, the Laplace mechanism is centralized, in that the data aggregator receives raw, unmodified data and then adds noise to query responses. That is, the Laplace mechanism is meant for a scenario in which a data aggregator already has the "true" data, but wants to release the results of queries without violating the privacy of the people whose data it controls. It does this by adding random noise to the output of these queries. The noise in this case is sampled from the Laplace distribution Links to an external site., hence the name.

The key to the Laplace mechanism is the way that it chooses the scale of the distribution the noise is sampled from. The scale of the distribution is the "spread," how large the standard deviation is (how likely you are to see significant outliers). A scale that is larger means that on average there will be more noise, and more privacy, whereas a scale that is closer to 0 will be closer to the original value and therefore less private. In the Laplace mechanism, the scale is set as the sensitivity of the query over epsilon. Recall that a smaller epsilon (privacy budget) means more privacy, whereas a larger epsilon means less privacy.

For our purposes, we can just think of a query as a function that takes data and produces a numerical answer. The sensitivity of the query is the maximum amount that the query can differ if you give it data that is modified in a single entry. For example, let's say we want to count the number of entries in a vector that are greater than 12.6. Think about two hypothetical databases (which we can think of right now as just vectors). These databases (vectors) are neighbors, meaning that they differ in exactly one entry. x and y in the cell below are examples of a set of neighboring databases.

x = np.array([0, 15, 26, 35, -10, 12])

y = np.array([12.7, 15, 26, 35, -10, 12])

The difference in answers between running the query on x and y is 1. A hypothetical attacker could use information like this (or more realistically, a series of slightly different queries) to recover information about people in the database.

Crucially, the most that the output of this query could be altered given any database z is by taking an entry of z that is above 12.6 to be below 12.6, or taking an entry below 12.6 to be above 12.6. In either case, this would only change the output of the query by at most 1, so the sensitivity of this query is 1. Therefore, the Laplace mechanism would add to the output of this query a value drawn from a Laplace distribution with scale 1/eps.

Write-up Question 4 (10 points): Implement the Laplace mechanism for the same (un-noised) counting query as in Question 1 from Task 1. Plot the distribution of answers as epsilon decreases for epsilon values 10, 1, 0.1 and 0.01. Scipy contains a method for generating Laplacian noise through the laplace Links to an external site.class. Include this plot in your write-up. No prose is needed for this question.

RESTATED Write-up Question 4 (10 points): Implement the Laplace mechanism for a counting query. For instance, inspired by the first task, imagine that you again have a group of 1,000 people and want to measure the number of people who are "yes" responses to a binary question. You can assume that the true number is whatever you want. This time, you won't be using randomized response, but instead differential privacy. Create either a histogram or a box plot of the distribution of answers for epsilon values 10, 1, 0.1 and 0.01, simulating 100 different times each of applying differential privacy to the true answer. Of course, in an actual situation, you'd only calculate the true answer and then add noise via differential privacy a single time, but this will let you see the variance introduced from adding noise. Scipy contains a method for generating Laplacian noise through the laplace Links to an external site.class. Include this histogram or box plot in your write-up. No prose is needed for this question.

Write-up Question 5 (5 points): Examine the graph you just made. What do you notice about the distribution of answers as epsilon decreases? What happens when epsilon is quite small (e.g., less than 0.01)? Does this pose any problems?

Write-up Question 6 (5 points): For each of the points in the graph you just plotted (corresponding to a set of 100 trials), look at the averages. How much do they differ from the true count? What is the standard deviation for each value of epsilon? What does this suggest about repeated queries under the differential privacy framework? How can a data curator/aggregator defend against repeated queries?

RESTATED Write-up Question 6 (5 points): For each of the values of epsilon in Question 4 (each containing a set of 100 trials of adding differentially private noise with that epsilon to the true answer), look at the averages. How much do they differ from the true count? What is the standard deviation for each value of epsilon? What does this suggest about repeated queries under the differential privacy framework? How can a data curator/aggregator defend against repeated queries?

Counting is useful, but sometimes we also need answers to other questions. Implement the Laplace mechanism for computing the mean of a real valued list. For this you will need to derive the sensitivity of the mean (i.e., how much a query may vary given a difference of one entry). To do this, you will need to add as a parameter the allowable range of the list, the maximum and minimum values that are allowable. In implementing this mechanism, think carefully about how to handle data that is out of the specified range.

Write-up Question 7 (7 points): First, briefly explain how you compute the sensitivity for the mean of a real valued list. Reflecting on this approach, if you drop data outside of the range, how would that affect the sensitivity analysis? Could this inadvertently reveal information meant to remain private?

RESTATED Write-up Question 7 (7 points): First, briefly explain how you compute the sensitivity for the mean of a real valued list. Give a formula (including relevant, defined variables). Your sensitivity calculation should assume that you are dropping a row of the dataset.

Write-up Question 8 (19 points): Using the data in the income data set, compute the differentially private average age with different bounds. Using epsilon = 0.1, plot the average difference between the true answer and the differentially private answers for feasible points in the grid formed by minimum and maximum age. Include this plot in your write-up. No prose is needed.

RESTATED Write-up Question 8 (19 points): Using the data in the income data set, compute the differentially private average age with different bounds (meaning different minimum and maximum ages). For this question, we want you to make a heatmap. In your heatmap, one axis should be the minimum age (use integers) and the other axis should be the maximum age (again, integers). Using epsilon = 0.1, plot the difference between the true answer and the differentially private answers for feasible points in the grid formed by minimum and maximum age. Feasible refers to the minimum age needing to be less than or equal to the maximum age, as well as all ages corresponding to reasonable human lifespans. Each cell of the heat map shows the difference from the real answer. Only run this simulation once, not many times, since of course if you run it many times it will average out the differences per cell. Include the heatmap in your write-up. No prose is needed.

Write-up Question 9 (4 points): Look at magnitude and direction of the error in the plot. What does this suggest about choosing reasonable cutoffs when handling differentially private data? How might one go about minimizing this sort of error? In interpreting the graph, it may be useful to also plot a histogram of the age.

RESTATED Write-up Question 9 (4 points): Look at magnitude and direction of the error in the plot. What does this suggest about choosing reasonable cutoffs when handling differentially private data?

Task 3: Exploration: Online Tracking (24 points)
Later this quarter, we'll be discussing the online tracking ecosystem. To understand the prevalence of tracking a bit more directly, download and install either the Ghostery Links to an external site.or Privacy Badger Links to an external site.browser extension on a web browser you use regularly. Either of these extensions will let you see which domains can directly track your visit to a particular webpage based on the outgoing HTTP requests as you load the page. Spend about 10 minutes visiting a variety of different websites you regularly visit and see which companies are tracking you on each site.

Write-up Question 10 (12 points): Write at most a paragraph describing what you experienced visiting webpages with either Ghostery or Privacy Badger showing you who is tracking you. Your answer should (briefly) respond to all of the following points. On what sites do you seem to encounter the largest number of trackers, and on what sites do you seem to encounter the least? What did you find most surprising from using your chosen tool to see who is tracking you?

Write-up Question 11 (12 points): Write at most a paragraph commenting on the tool's interaction design; don't forget to mention which one you are using. Your answer should (briefly) respond to all of the following points. Who might be the intended users of this tool from what you can gather? What information does the tool seem to focus on conveying, and do you think it's the right information to convey to those users? If you were redesigning the tool, what might you consider doing differently?
