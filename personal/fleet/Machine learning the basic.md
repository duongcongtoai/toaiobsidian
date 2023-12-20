Created: 2023-11-08
# Derivative and gradient
gradient is generalization of derivative
f(x) can be generalized in a function of error lost
the math problem of finding the top/bottom of the bell curve is now generalized as the problem of finding point where combination of the hyper parameters(ax1,ax2,ax3...) associaated to the lowest value of the error lost function

Take small steps (learning rate), given this set of hyper parameters, find the next direction that can lead to better set of hyper parameters (where the computed error lost value is smaller)
## Mini-batch stochastic gradient descent
Computing the gradient value given the set of hyper parameters (ax1,ax3,..,axn) with n is the number of future for all of dataset in reality is expensive => sample data into smaller batches and only compute gradient for that batch. Then take the average number as the representative gradient value to apply gradient descent
## Generalization 
the characteristic of the model that it can apply its stable correctness to the never before observed data
If not, it may has a potential of overfitting 
To avoid overfitting, partition data into 3 types:
## Feature engineering

The process of extract raw data into value of usable feature vectors
say data has 1k sample, feature vector of feature f1 is an array with 1k items

for each model m1:
- training data: the process of apply(learning_rate,batch_size) to find the optimal set of hyper parameter associated with the lowest error lost (training_error_lost)
- validation data: validate the model m1 with the validation data to check the symptom of overfitting

Choose the best model M to test with the test dataset


## References
1. 
tags: 