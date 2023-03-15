# Linear Regression Model Guide - theory part


## Introduction

Recently, I review the machine learning course of Andrew ng in Coursera. Surprisingly, I can still learn a lot, so I decided to write some posts👍.

To talk about linear regression, we must first have a basic understanding of what is machine learning. What is machine learning? abstractly speaking, **machine learning is learning a function**:
$$
f(input) = output
$$
where $f$ refers to the specific machine learning model. **Machine learning is a methodology for automatically mining the relationship between input and output**. Sometimes we find it hard to define a specific algorithm to solve some problems, and this is where machine learning shines, we can let it learn and summarize some patterns from data and make predictions. This is also where it differs from traditional algorithms (binary search, recursive, etc.). One has to admit that machine learning is fascinating by definition, and it *seems* to provide a viable framework for solving all intractable problems. It just so happens that many real-life problems are so hard that solving them with traditional algorithms is impossible.

> 📒 That's why I always feel that **every programmer should know some machine learning/deep learning**, it is a great tool for us to solve problems💪

Linear regression is one of the classic machine learning models. In my opinion, it can be regarded as the counterpart of ["Hello world"](https://en.wikipedia.org/wiki/%22Hello,_World!%22_program) in the machine learning world.

In the previous formula, $input$ is the so-called **feature** in machine learning, which is often represented by the symbol $x$. And $output$ can be roughly divided into two categories: class/category (Classification problem) and number (Regression problem).

> 📒 How to understand features? The so-called **features are characteristics that are highly correlated with the target to be predicted**. *Take predicting housing prices as an example. The housing price is obviously related to the house area, distance to the downtown, etc. The house area here is a feature*. In machine learning, we can analyze the importance of different features to see which ones are highly correlated with the output, or use our domain knowledge (Domain knowledge) to choose good features wisely. But in general, **machine learning still requires us to spend a lot of time spotting good features**. This is the drawback of machine learning, and this shortcoming is greatly alleviated in deep learning. Of course, that is not the topic today.

> 📒 Machine learning **cannot magically understand** the various formats of $input$ you provide, *such as images, text, videos, etc.*. In machine learning, **$input$ is often processed as numbers**, so that they can be learned by various machine learning algorithms. Some features are numbers themselves, *such as the house area we previously talked about*. When the input is not transformered into numbers, we needed to invent some procedure to do so, *such as using a word embedding vector model to represent text, etc.*, and I will not go into details here. **Let's assume that the input has been processed into numbers below**.

The topic of this post, the linear regression model, belongs to the regression model in the **supervised learning domain**. It is a simple model, but it can explain a lot of machine learning ideas, which makes it a perfect choice to get started. Without further ado, let's get started :)

> 📒 This article assumes that you have a basic understanding of matrix/vector calculus, *for example, you need to know the definition of matrix multiplication, how to calculate derivatives, etc*.

## Linear regression model

As the name suggests, it consists of two parts:
- Linear. *Taking a two-dimensional plane as an example, $y=kx+b$ is linear, for it is a straight line when drawn, but $y=x^2$ is not, because it is drawn as a curve*
- Regression because it fits the definition of a regression problem in **machine learning** - predicting a value of **unlimited range**

## Categories

> 📒 To reduce the visual noise, the following notation $f(x)$ omits the subscripts $w,b$ for simplicity. I.e. $f(x)=f_{w,b}(x)$

> 📒 Sometimes you may see some books or blogs use $h_\theta$ to represent the model(h means hypothesis), I think $f(x)$ as it is more concise and intuitive.

The linear regression model can be divided into the following categories:

### Univariate linear regression model

Univariate means that your model only accepts a single feature:
$$
f(x)=wx+b
$$
In machine learning world, we call $w$ and $b$ as weight and bias

> 📒 Note: $\theta$ is a column vector, logically it should be represented by $\vec \theta$, but for the sake of brevity, I have omitted the arrow.

> 📒 The equation above can also be written in the form of vector multiplication. Let $\theta=[b, w]$ represent the parameters of the model, and rewrite x as $\vec x=[1, x]^T$. According to the knowledge of vector multiplication, we can infer that $\theta^T\vec x=wx+b$

### Multiple linear regression model

Multiple means that your model accepts multiple features:
$$
f(x)=w_1x_1+w_1x_2+...w_nx_n+b
$$
Where $x_i$ is the ith feature, there are $n$ features in total, for this, we need to learn a weight $w_i$ for each feature

> 📒 Similarly, we can choose to write in the form of vector multiplication. Let $\theta=[b, w_1, w_2, ..., w_n]$ and $\vec x=[1, x_1, x_2, ..., x_n]^T$. You will be surprised to find that **univariate linear regression and multivariate linear regression have a unified form - $f(\vec x)=\theta^T\vec x$**. This will bring great convenience when calculating the gradient later.

## How to train this model

### Cost function

After defining the linear regression model, we need to measure its prediction quality. Obviously, we wish to minimize the "distance" between the predicted value and the actual value. In machine learning, we will specify a cost function to measure the "distance". **The lower the cost, the better**. When the cost of the model on the validation set is minimized to a small value and no longer changes significantly, we think that the model has converged, and the best model is obtained at this time

> 📒 In machine learning, the data is usually divided into training/validation/test set, the model learns on the training set, verifies and adjusts parameters on the validation set, and performs generalization verification on the test set finally (**The test set contains unseen data**). It is incorrect to directly adjust parameters on the test set by only using the division method of training/test set. The test set is used **if and only if** after you finish training and tuning your model

The most commonly used cost function of the linear regression model is the mean square error function(MSE). Its formula is as follows:

$$
MSE=\frac{1}{2m}\sum^m_{i=1}(\hat y^{(i)} - y^{(i)})^2
$$

where
- $m$ represents the number of samples
- The superscript ${(i)}$ is different from exponent marks. ${(i)}$ indicates the predicted/true value of the $i$th sample
- $\hat y$ represents the prediction of the linear regression model, and $y$ represents the original true value

> 📒 I use notations that conforms to machine learning conventions~

> 📒 Linear regression model **does not have to use MSE though**, you can also use the absolute value error(MAE). Use MAE when there are many outliers in the training set, which can reduce the sensitivity to them.

### Training use gradient descent

As aforementioned, when the cost of the model on the validation set is minimized and does not change significantly, we get the best model. But how to make the model continuously improve the prediction and reduce the cost? 🤔 The Gradient descent algorithm is a general practice

Don't be fooled by the name, the gradient descent algorithm is not that mysterious. Let's recap what we have learned so far:
- The prediction $\hat y$ of the model is related to its parameters, *if it is a univariate linear regression model, it is related to $w$ and $b$*
- The cost function is related to the prediction of the model because we cannot change the real value of the data

So **The cost function is a function about the model parameters**. In machine learning, it is usually denoted by $J$.

Consider first simple univariate linear regression when it uses MSE as the cost function:

$$
J(w, b) = \frac{1}{2m}\sum^m_{i=1}(f(x^{(i)})-y^{(i)})^2 = \frac{1}{2m}\sum^m_{i=1}(wx^{(i)} + b - y^{(i)})^2 
$$

**By changing the values of $w$ and $b$, the cost $J(w,b)$ will change accordingly**, so **improving the linear regression model is adjusting these two parameters so that $J(w, b)$ reaches its minimum**. In the language of mathematics, solving the optimal model is solving the minimum point of the function $J(w,b)$

> 📒 Recall that $m$ is the number of samples in the training set. More strictly speaking, the above gradient descent formula is batch gradient descent, that is, **every calculation of the gradient uses samples from the entire training set**. When the training set is very large, this method cannot scale well. At this time, we need to use stochastic gradient descent.

> 📒 If you are good at mathematics, it is not difficult for you to find the minimum value of the above formula. But **this is because the linear regression model is relatively simple**, when the model becomes more complicated, it will be more difficult to solve it with mathematical methods. **In practice, we use the gradient descent algorithms all the time in the machine learning world** (reinforcement learning is an exception, for the goal of reinforcement learning optimization is generally not a differentiable function)

Let us **forget $b$ temporarily and only consider $w$**, then $J$ is a function about $w$ at this time. We can take $w$ as the horizontal axis, and $J(w)$ as the vertical axis, which gives us the following plot

{{< figure src="/img/linear_regression_tagent.png" width="70%" >}}

> 📒 Note that the above diagram does not strictly follow the definition of $J(w)$. For convenience, I directly draw $y=\frac{1}{2}x^2$ here. But **the shape should be similar**, which can be used to understand the gradient descent algorithm.

It is trivial to find out from the figure that $J(w)$ reaches its minimum when $w=0$. Assuming that the model's current parameter $w$ is $5$, we can calculate the cost as $J(5)=12.5$. How do we update $w$ to minimize the cost? **The solution is to let $w$ update along the **opposite direction** of the gradient (the red line in the figure is the tangent at the point $w=5$)**, it is not difficult to compute the gradient/derivative at this point, which is $5$, $w$ should be reduced, so it should minus this gradient (that's why we say opposite direction). We also introduce **learning rate $\alpha$ to control the step size**, which give us the update formula $w \leftarrow w - \alpha \cdot 5$

> 📒 Intuition: Imagine yourself standing at the point of $w=5$ and want to reach the minimum. If the learning rate $\alpha$ is too large, your new position $w$ may be less than $0$. You will find that you suddenly crossed the minimum. Although you can try to update again through gradient descent, at this time you will often find that the loss of your model has been changing and cannot converge. **The learning rate and the gradient jointly control the magnitude of each parameter update**

> 📒 We concluded how to update when only considering $w$ - **along the opposite direction of the gradient**. **This conclusion still holds when considering more parameters**, but that involves the knowledge of solving partial derivatives and even vector derivatives, which also makes visualizing the $J$ more difficult, so I won’t go into details here. **Just remember that the update of the model always moves in the opposite direction of the gradient**👻

In the univariate linear regression model, the gradient update formula is as follows:
$$
w \leftarrow w - \alpha \cdot \frac{\partial}{\partial w}J(w,b)
$$

Don't forget the $b$, which should also be updated:
$$
b \leftarrow b - \alpha \cdot \frac{\partial}{\partial b}J(w,b)
$$

Let's calculate the gradients:
$$
\begin{aligned} 
\frac{\partial}{\partial w}J(w,b)&=\frac{\partial}{\partial w}\frac{1}{2m}\sum_{i=1}^m(f(x^{(i)})-y^{(i)})^2 \\\\\\
&= \frac{\partial}{\partial w}\frac{1}{2m}(wx^{(i)}+b-y^{(i)})^2 \\\\\\
&= \frac{\partial}{\partial w}\frac{1}{m}\sum_{i=1}^m(f(x^{(i)})-y^{(i)})x^{(i)} 
\end{aligned} 
$$
$$
\begin{aligned} 
\frac{\partial}{\partial b}J(w,b)&=\frac{\partial}{\partial w}\frac{1}{2m}\sum_{i=1}^m(f(x^{(i)})-y^{(i)})^2 \\\\\\
&= \frac{\partial}{\partial b}\frac{1}{2m}(wx^{(i)}+b-y^{(i)})^2 \\\\\\
&= \frac{\partial}{\partial b}\frac{1}{m}\sum_{i=1}^m(f(x^{(i)})-y^{(i)})
\end{aligned} 
$$

If it is a multiple linear regression, we need to solve each $\frac{\partial }{w_i}J(w_1,w_2, ...,b)$. It is very inconvenient. **So, let's derive the gradient in vector form**

In vector form, the cost function MSE is written as:

$$
J(\theta) = \frac{1}{2m}(X\theta - \vec{y})^T(X\theta - \vec{y})
$$

where
- $X$ is the input matrix. **The uppercase letters are commonly used to represent the matrix**. Each of its rows is the values of features of each sample - $(x^{(i)})^T$, and the dimension of $X$ is $(m, n +1)$
     - As aforementioned, we got $n$ features, but we also add an intercept term so we got $n+1$ rather than $n$. 
     - $x^{(i)}$ needs to be transposed because it is originally a column vector
     - $\theta^T\vec x$ for a single sample, matrix multiplication $X\theta$ for $m$ samples :)
- $\theta$ as we said before is $[b, w_1, w_2, ..., w_n]$, with length $n + 1$
- So $X\theta$ is the predicted value of the model, according to the knowledge of matrix multiplication, we will get a vector of length $m$, representing the predictions for each sample
- $X\theta - \vec y$ is the error of each prediction
- By the way, assuming $\vec a$ is a column vector, **$\vec a^T\vec a$ always gets a scalar sum of squares** of each element. If you define $\vec a=X\theta -\vec y$ you roughly get the right handside of the equation.

Next, let's try to derive the gradient of this formula, **this requires you to have a certain knowledge of vector/matrix calculus, you can choose to skip it🔮 now**. ~~But I highly recommend you to take a look, because there are quite a lot of matrix calculus in machine learning~~

$$
\begin{aligned} 
\frac{\partial}{\partial \theta}\ J(w,b) 
&= \frac{\partial}{\partial \theta}\ \frac{1}{2m}(X\theta - \vec{y})^T(X\theta - \vec{y}) \\\\\\
&= \frac{1}{2m}\frac{\partial}{\partial \theta}\ (\theta^TX^T - \vec{y}^T)(X\theta - \vec{y}) \\\\\
&= \frac{1}{2m}\frac{\partial}{\partial \theta}\ (\theta^TX^TX\theta - \theta^TX^T\vec y - \vec y^TX\theta + \vec y^T\vec y) \\\\\
&= \frac{1}{2m}\frac{\partial}{\partial \theta}\ (\theta^TX^TX\theta - \theta^T(X^T\vec y) - (X^T\vec y)^T\theta + \vec y^T\vec y) \\\\\\
&= \frac{1}{2m}\frac{\partial}{\partial \theta}\ (\theta^TX^TX\theta - 2\theta^T(X^T\vec y) + \vec y^T\vec y) \\\\\\
&= \frac{1}{2m}(\frac{\partial}{\partial \theta}\ (\theta^TX^TX\theta) - 2\frac{\partial}{\partial \theta}(\theta^TX^T\vec y))) \\\\\\
&= \frac{1}{2m}(2X^TX\theta - 2X^T\vec y)) \\\\\\
&= \frac{1}{m}(X^TX\theta - X^T\vec y)) \\\\\\
&= \frac{1}{m}X^T(X\theta-\vec y)
\end{aligned} 
$$

Explanations:
- The shape of $X^T$ is $(n+1, m)$, and the shape of $\vec y$ is $(m, 1)$, so $X^T\vec y$ will give us a column vector with $(n +1, 1)$. **Note that in linear algebra, when $\vec a$ and $\vec b$ are both column vectors, $\vec a^T\vec b=\vec b^T\vec a$ holds**. So $\theta^T(X^T\vec y) = (X^T\vec y)^T\theta$ is satisfied.
- For the derivation in several other places, I recommend reading the [Cheatsheet](http://www.gatsby.ucl.ac.uk/teaching/courses/sntn/sntn-2017/resources/Matrix_derivatives_cribsheet.pdf). Of course, if you are interested in this aspect, it is recommended to try to prove them by yourself
     - For example, to compute the $\frac{\partial}{\partial \theta}(\theta^TX^T\vec y))$, we can let $\vec b=X^T\vec y$, which just corresponds to $\frac{\partial }{\partial \theta}\theta^T\vec b=\vec b$ in cheatsheet

> 📒 The vector form also brings great convenience when implementing this model. We can use the mature [Numpy](https://numpy.org/) for calculations instead of writing a for loop by computing one by one.

## Wrap up

We have finished the theoretical part of the linear regression model. This post introduces univariate linear regression and multivariate linear regression. The former can be regarded as a special case of the latter. Finally, we unified the equations in vector form, and derive the gradient when MSE is used as the cost function.

I originally wanted to put the code here, but I found out that this blog is already very long. It will be a better choice to put the implementation part in another post🙌


