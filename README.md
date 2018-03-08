# MATLAB code for training deep belief networks. 

Implementation is based on the following references: 

1. Hinton, G., Osindero, S., & Teh, Y.-W. (2006). A fast learning algorithm for deep belief nets. Neural Computation, 18(7), 1527–1554.
2. Hinton, G. (2010). A practical guide to training restricted Boltzmann machines. Momentum, 9(1), 926.
3. Palm, R. B. (2012). Prediction as a candidate for learning deep hierarchical models of data (Master's thesis).
4. Larochelle, H., & Bengio, Y. (2008). Classification using discriminative restricted Boltzmann machines. In Proceedings of the 25th international conference on Machine learning (pp. 536–543). ACM.

I wrote this code as part of my undergraduate project several years ago. It was an attempt to reproduce results from  Hinton's 2006 paper ["A fast learning algorithm for deep belief nets"](http://www.cs.toronto.edu/~hinton/absps/fastnc.pdf). Inspired by Geoffrey Hinton's [neural network simulation](http://www.cs.toronto.edu/~hinton/adi/), a visualization of the generative learning process  of a DBM, I also attempted to produce a similar sort of visualization. When it came to implementing the softmax units in the top-most "associative memory" layer of the DBN, I found the descriptions in Larochelle 2008 to be very useful - some of which are reproduced below.


The following samples were generated by randomly initializing the visible units in the associative layer and "clamping" the softmax units to one of the target classes. After each iteration of Gibbs sampling, a downward pass through the network was performed to generate a sample, which was saved to an image file. The animations show how the samples evolve over 200 iterations of Gibbs sampling. It's interesting to see how quickly the Gibbs sampling converges.

<div style="text-align: center;" markdown="1">

![](/images/0.gif)
![](/images/1.gif)
![](/images/2.gif)
![](/images/3.gif)
![](/images/4.gif)
![](/images/5.gif)
![](/images/6.gif)
![](/images/7.gif)
![](/images/8.gif)
![](/images/9.gif)
</div>

Softmax Units
--------------------
To model the joint the distribution of input data and associated target classes, the visible layer contains an additional group of softmax units, used to represent the ![](https://latex.codecogs.com/gif.latex?k) alternative target classes in the MNIST training data. 

For the "associative memory" layer of the DBN, the energy function includes additional terms for the ![](https://latex.codecogs.com/gif.latex?%5Cmathbf%7BU%7D%24%20and%20%24%5Cmathbf%7Bd%7D) parameters of the softmax units, which respectively represent the softmax-to-hidden connections and the softmax biases.

![](https://latex.codecogs.com/gif.latex?E%5Cleft%28y%2C%20%5Cmathbf%7Bx%7D%2C%20%5Cmathbf%7Bh%7D%20%5Cright%29%20%3D%20-%5Cmathbf%7Bh%7D%5ET%20%5Cmathbf%7BW%7D%20%5Cmathbf%7Bx%7D%20-%20%5Cmathbf%7Bb%7D%5ET%20%5Cmathbf%7Bx%7D%20-%20%5Cmathbf%7Bc%7D%5ET%20%5Cmathbf%7Bh%7D%20-%20%5Cmathbf%7Bd%7D%5ET%20%5Cmathbf%7By%7D%20-%20%5Cmathbf%7Bh%7D%5ET%20%5Cmathbf%7BU%7D%20%5Cmathbf%7By%7D)

Training the the associate memory layer of the DBN is the same as training an RBM with standard binary units, but the conditional probabilities have a slightly different form.

![](https://latex.codecogs.com/gif.latex?p%5Cleft%28y%20%5Cmid%20%5Cmathbf%7Bh%7D%20%5Cright%29%20%3D%20%5Cfrac%7Be%5E%7Bd_y%20&plus;%20%5Csum_jU_%7Bjy%7Dh_j%7D%7D%7B%5Csum_%7By%5E*%7De%5E%7Bd_y%20&plus;%20%5Csum_jU_%7Bjy%5E*%7Dh_j%7D%7D)

The conditional probability of a particular hidden unit being active includes additional terms for the softmax units.

![](https://latex.codecogs.com/gif.latex?p%5Cleft%28h_j%20%3D%201%20%5Cmid%20y%2C%20%5Cmathbf%7Bx%7D%20%5Cright%29%20%3D%20%5Cmathrm%7Bsigm%7D%7B%5Cleft%28%20c_j%20&plus;%20U_%7Bjy%7D%20&plus;%20%5Csum_i%20W_%7Bji%7Dx_i%20%5Cright%29%7D)

To choose the most probable class under the model, the conditional distribution can be computed exactly for reasonable numbers of classes. In practice, the unnormalized probabilities can be used to determine the most probable class.

![](https://latex.codecogs.com/gif.latex?p%5Cleft%28%20y%20%5Cmid%20x%5Cright%29%20%3D%20%5Cfrac%7Be%5E%7Bd_y%7D%5Cprod_%7Bj%3D1%7D%5En%20%5Cleft%28%7B1%20&plus;%20e%5E%7Bc_j%7D%20&plus;%20U_%7Bjy%7D%20&plus;%20%5Csum_i%20W_%7Bij%7Dx_i%7D%5Cright%29%7D%7B%5Csum_%7By%5E*%7De%5E%7Bd_%7By%5E*%7D%7D%5Cprod_%7Bj%3D1%7D%5En%20%5Cleft%28%201%20&plus;%20e%5E%7Bc_j%20&plus;%20U_%7Bjy%5E*%7D%20&plus;%20%5Csum_i%20W_%7Bji%7D%20x_i%7D%5Cright%29%7D)