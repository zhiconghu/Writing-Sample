# Optimisation Project: Elective Selection by Preferences

This is an Optimisation Project for AM13 - Decision Analytics & Modelling Couse in London Business School. In this project, we aim to tackle the problem of MAM student 
choosing their electives accounting for subject interest, number of credits, GIFTs, and preferred experiences.

# Problem Defintion

LBS students are faced with an important decision during their master's programme: choosing their electives. From a pool of 60 courses, students have to identify where 
their interests lie, what they want their schedule to look like in the upcoming months, what type of evaluation they want to be scored on, whether to opt for more technical 
or managerial courses, what they value from courses at LBS. And all of this must be decided by long and tedious reading and analysing webpages with information regarding 
each course. It can be difficult to navigate the different topics, ratings, whilst still getting the right number of credits and not having any clashing schedules. 

To make this process a little simpler, adaptable and less hour intensive, we have decided to build an **optimisation tool that includes student's preferences and inclinations 
in the elective selection process**. To build it, the tool considers:

* The total amount of extra credits allowed for each student (whether they take a GIFT or want to max out credits or not)
* The subject area of each course
* The term and type of scheduling for each elective
* The weight that students give to each of the 7 evaluation scores of electives captured in previous years

From those data points, the model will optimise to choose electives with the highest average evaluation scores given the weighst, and give a suggestion to students about which 
electives to choose.

# Data

First, we created a comprehensive data set that allowed us to carry out the optimisation. This data was gathered mostly from the Enrolment Management System webpage 
*(https://ems.london.edu/CourseDirectory)* and includes all 60 courses offered to Master's in Analytics and Management students in the 2021-2022 academic year. 

The columns describe:

* **Elective ID**: The columns 'Name', 'Number' and 'Stream' help identify the specific elective that is being referred to. 
* **Credits**: The 'Credit' column indicates whether they are a 6 or 11 credit course. 
* **Subject and Faculty**: They describe the area of study and teaching professor for each course. 
* **Schedule**: The columns 'Term'. 'Schedule', 'Weekday', 'Start Time', 'End Time', 'Block Week Number' and the following 24 columns are there to help us identify when each
of the electives is happening in order to build a functional schedule. 
* **Evaluation scores**: 7 questions posed to students after finishing their elective modules evaluate the student satisfaction, and are saved on the data set for evaluation.
The mean, maximum and minimum values are shown hereafter for all seven. 

# Model Development

The *Elective Selection Problem* is an optimisation problem aims to build a tool to simplify the selection process for MAM students simple and less hour intensive that includes 
student’s preferences and inclinations. The algorithm must consider all the factors to optimise the chosen electives with the highest evaluation score. In the following part,
we will formulate our optimization problem.

## Decision Variable

Our decision variables are whether to take an elective or not, so it would be a binary variable for each course:

$$ x_{course}$$

where x is binary.

## Must-Have Constraints

These are constraints that should be faced by any individuals regardless of their preferences.

### Stream Constraint

This constraint describes the fact two different streams of the same elective cannot be selected, i.e. "Advanced Competitive Strategy A" and "Advanced Competitive Strategy B" 
cannot be both taken. To create this elective, we differentiate between elective 'name' and elective 'stream'. 

The courses above have multiple streams for the same elective name. We can then set constraints on those by locating their index and setting the sum of these indices to less 
than or equal to 1, i.e. only one of them can be selected. In mathematical notation: 

$$ \sum_{stream}x_{stream} \leq 1 $$

where $x_{stream}$ are decision variables of the electives that are the same electives but are just of different streams.

### Timetable Constraint

Following, we look for clashes in classes and set constraints so that our selected electives do not have a clashing schedule. 

Block weeks are hosted in weeks that are different from the rest so we can treat block weeks seperately from the rest electives. For other electives of other schedules, we 
create a sparse matrix that has binary values, indicating at what time these electives are held. Lastly, we also set a constraint on the terms, we cannot take more than 3 
electives that are in the same term to prevent too much overload.

**Block Week**

We cannot have a clash in block weeks, i.e. take two electives that are on the same block week. Using a for loop we evaluate all elective with a block week type teaching 
method. The constraint, in mathematical notation: 

$$ \sum_{block}x_{block} \leq 1 $$

where $x_{block}$ are decision variables of the electives that have same block week number.

**Scheduling**

We cannot have a clash in scheduled time of classes. We will make use of our sparse matrix here. It is important to note that we can have classes with the same schedule in the 
case that they are in different terms (spring or summer). The mathematical expression for the created constraint: 

$$ \sum_{n \in allCourses}x_n*s_{tc}\leq 1 $$

where $s_{tc}$ is a vector in the one-hot matrix representing the schedules of our classes, column names are all possible schedule times (separated by time of day, weekday and term).

**Term**

We cannot have more than 3 electives in a single term. The mathematial expression for the created constraint:

$$ \sum_{term}x_{term} \leq 3 $$

where $x_{term}$ are decision variables of the electives that are in the same term.

### Credit Constraint

Here, we are setting the credit constraint. As a basis, we are setting it as not taking GIFTs nor the extra credit, therefore, we will be limited to 44 total credits. Furthermore, as 
we are not taking the extra credit, we will only be able to take 11 credit courses.

The mathematial expression for the created constraint:

$$ \sum_{n \in allCourses}x_{n} * credit_{n} \leq 44 $$

where $credit_n$ is the credit of elective $n$.

We also need to set a constraint for not being able to select a 6 credit elective unless we choose to take the extra 6 credit. This constraint can be changed based on user preference, 
it will be no in default, and in mathematical terms:

$$ \sum_{extra}x_{extra} = 0 $$

where $x_{extra}$ are decision variables of the 6 credit electives.

## Perference Constraints

The following part allows for addition constraints based on user preferences. There will be 4 choosable constraints:

* Whether to take GIFTs
* Whether to 6 Extra Credit electives
* Subject Preferencs
* Experience Preferences (Adding weights on particular questions)

### GIFTs Constraint

We can choose whether to take the GIFTs by change the variable "gifts" to "Yes" or "No".

### Extra Credit Constraint

Likewise, we can choose whether to take the extra 6 credit electives by change the variable "extra" to "Yes" or "No".

### Add Constraints based on Credit Preferences

This will change our credit constraint and whether we can select 6 credit course constraint based on selected preferences. Base on our preference in whether to take GIFTs or the extra 
6 credit elective, we will be setting different constraints:

If we are taking GIFTs and the extra 6 credit elective, we will be setting the following constraints:

$$ \sum_{n \in allCourses}x_{n} * credit_{n} = 39 $$

$$ \sum_{extra}x_{extra} = 1 $$

where $credit_n$ is the credit of elective $n$ and $x_{extra}$ are decision variables of the 6 credit electives.

If we are taking GIFTs and not taking the extra 6 credit elective, we will be setting the following constraints:

$$ \sum_{n \in allCourses}x_{n} * credit_{n} = 33 $$

$$ \sum_{extra}x_{extra} = 0 $$

where $credit_n$ is the credit of elective $n$ and $x_{extra}$ are decision variables of the 6 credit electives.

If we are not taking GIFTs but taking the extra 6 credit elective, we will be setting the following constraints:

$$ \sum_{n \in allCourses}x_{n} * credit_{n} = 50 $$

$$ \sum_{extra}x_{extra} = 1 $$

where $credit_n$ is the credit of elective $n$ and $x_{extra}$ are decision variables of the 6 credit electives.

If we are not taking GIFTs and not taking the extra 6 credit elective, we will be setting the following constraints:

$$ \sum_{n \in allCourses}x_{n} * credit_{n} = 44 $$

$$ \sum_{extra}x_{extra} = 0 $$

where $credit_n$ is the credit of elective $n$ and $x_{extra}$ are decision variables of the 6 credit electives.

### Subject Preference Constraint

Here, we can specific our subject preference to get more personalized elective selections. For our selected subject preference, we will assign at least half of the credits to 
that selected subject. In mathematical notations:

$$ \sum_{subject}x_{subject} * credit_{n} \geq \frac{TotalCredit}{2} $$

where $x_{subject}$ are the decision variables of electives in our preferred subject.

## Objective Function

We are also able to select our preferred experience from our selected electives. This will affect our objective function that we are maximising. The experiences can be quantified 
by the evaluation questions asked:

1. Overall, how much do you think you have learnt from the course?
2. How well do you believe the course met its stated objectives?
3. How would you rate the overall effectiveness of the faculty?
4. How would you rate the timeliness of the feedback on course work/assignments from the instructors?
5. To what extent did the faculty provide useful feedback on course work/assignments?
6. How well did the faculty manage high quality standards for class participation?
7. How much previous knowledge of the subject did you have?

For example, if we enjoy class participation in class and we want electives that allows for that, we can only optimize based on the evaluation score of Q6 in our objective function.

We group our questions into 4 different experiences that we can choose from:

* Learning Experience: Q1, Q2, Q3
* Coursework: Q4, Q5
* Class Participation: Q6
* Prior Knowledge: Q7

With the question weight dictionary that are specify by our experience preference, we will then define our objective function based on the evaluation score. We will be maximising 
the following equation:

$$ \sum_{n \in courses}  x_{n} \left[ \left(\sum_{Q1 - Q6} weightDictionary * evaluationScore \right) - weightDictionary_{Q7} * evaluationScore_{Q7} \right] $$

where $weightDicitionary$ is obtained based on our experience preference.

We choose to minus evaluation score for Q7 because it is better that an elective requires less previous knowledge.

