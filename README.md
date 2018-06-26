生成对抗网络GAN
=============
# 一.GAN基本思想
> 生成式对抗网络GAN (Generative adversarial networks) 是Goodfellow 等在2014年提出的一种生成式模型。GAN 的核心思想来源于博弈论的纳什均衡。它设定参与游戏双方分别为一个生成器(Generator)和一个判别器(Discriminator), 生成器捕捉真实数据样本的潜在分布, 并生成新的数据样本; 判别器是一个二分类器, 判别输入是真实数据还是生成的样本。 
为了取得游戏胜利, 这两个游戏参与者需要不断优化, 各自提高自己的生成能力和判别能力, 这个学习优化过程就是寻找二者之间的一个纳什均衡。生成器和判别器均可以采用目前研究火热的深度神经网络.


# 二.GAN结构
> GAN的计算流程与结构如图：
![Image text](https://github.com/ShaoQiBNU/Generative_Adversarial_Nets/blob/master/images/1.png)

> 任意可微分的函数都可以用来表示GAN 的生成器和判别器, 由此,我们用可微分函数D和G来分别表示判别器和生成器, 它们的输入分别为真实数据x 和随机变量z。G(z)为由G生成的尽量服从真实数据分布pdata的样本。如果判别器的输入来自真实数据, 标注为1.如果输入样本为G(z), 标注为0。 
这里D 的目标是实现对数据来源的二分类判别: 真(来源于真实数据x的分布) 或者伪(来源于生成器的伪数据G(z)),而G的目标是使自己生成的伪数据G(z)在D上的表现D(G(z))和真实数据x在D上的表现D(x)一致.

# 三. GAN训练方法

> 判别模型： 希望真样本集尽可能输出1，假样本集输出0。对于判别网络，此时问题转换成一个有监督的二分类问题，直接送到神经网络模型中训练就ok。

> 生成网络：目的是生成尽可能逼真的样本。原始生成网络如何知道真不真？就是送到判别网络中。在训练生成网络的时候，需要联合判别网络才能达到训练的目的。什么意思？如果单单只用生成网络，那么后面怎么训练，误差来源在哪里？细想一下没有，但是如果把刚才的判别网络串接在生成网络的后面，这样就知道真假来，也就有了误差！！！所以对生成网络的训练其实是对生成-判别网络串接的训练，就像图中显示的那样。生成假样本，要把这些假样本的标签都设置为 1，也就是认为这些假样本在生成网络训练的时候是真样本。起到了迷惑判别器的目的！！！在训练这个串接网络的时候，一个很重要的操作就是不要判别网络的参数发生变化，也就是不让它参数发生更新，只是把误差一直传，传到生成网络那块后更新生成网络的参数，这样就完成来生成网络的训练。

> 论文里的公式如下：
![Image text](https://github.com/ShaoQiBNU/Generative_Adversarial_Nets/blob/master/images/2.png)

> 简单分析一下这个公式，整个式子由两项构成。x表示真实图片，z表示输入G网络的噪声，而G(z)表示G网络生成的图片。
D(x)表示D网络判断真实图片是否真实的概率（因为x就是真实的，所以对于D来说，这个值越接近1越好）。
D(G(z))是D网络判断G生成的图片是否真实的概率，G应该希望自己生成的图片“越接近真实越好”。也就是说，G希望D(G(z))尽可能得大。

> D的目的：D的能力越强，D(x)应该越大，D(G(x))应该越小，这时V(D,G)会变大，因此式子对于D来说是求最大(max_D)。
G的目的：G的能力越强，D(G(z))应该越大，D(x)应该越小，这时V(D,G)会变小，因此式子对于G来说是求最小(min_G)。

> 优化D，即优化判别网络时，没有生成网络什么事，后面的G(z)这里就相当于已经得到的假样本。优化D的公式的第一项，使得真样本x输入的时候，得到的结果越大越好，因为真样本的预测结果越接近1越好；对于假样本，需要优化的是其结果越小越好，也就是D(G(z))越小越好，因为它的标签为0。但是第一项越大，第二项越小，就矛盾了，所以把第二项改为1-D(G(z))，这样就是越大越好。
![Image text](https://github.com/ShaoQiBNU/Generative_Adversarial_Nets/blob/master/images/4.png)

> 优化G的时候，这个时候没有真样本什么事，所以把第一项直接去掉，这时候只有假样本，但是我们说这个时候是希望假样本的标签是1，所以是D(G(z))越大越好，但是为了统一成1-D(G(z))的形式，那么只能是最小化1-D(G(z))，本质上没有区别，只是为了形式的统一。之后这两个优化模型可以合并起来写，就变成最开始的最大最小目标函数了。
![Image text](https://github.com/ShaoQiBNU/Generative_Adversarial_Nets/blob/master/images/5.png)

> 训练两个模型的方法：单独交替迭代训练，算法过程如下：
![Image text](https://github.com/ShaoQiBNU/Generative_Adversarial_Nets/blob/master/images/3.png)
这里红框圈出的部分是我们要额外注意的。第一步我们训练D，D是希望V(G, D)越大越好，所以是加上梯度(ascending)。第二步训练G时，V(G, D)越小越好，所以是减去梯度(descending)。整个训练过程交替进行。

# 四. loss函数设定

> 由于tensorflow只能做minimize，loss function可以写成如下：
```python
D_loss = -tf.reduce_mean(tf.log(D_real) + tf.log(1. - D_fake))
G_loss = -tf.reduce_mean(tf.log(D_fake))
```

> 值得注意的是，论文中提到，比起最小化 tf.reduce_mean(1 - tf.log(D_fake)) ，最大化tf.reduce_mean(tf.log(D_fake)) 更好。

> 另外一种写法是利用tensorflow自带的tf.nn.sigmoid_cross_entropy_with_logits 函数：
```python
D_loss_real = tf.reduce_mean(tf.nn.sigmoid_cross_entropy_with_logits(
    logits=D_logit_real, labels=tf.ones_like(D_logit_real)))
D_loss_fake = tf.reduce_mean(tf.nn.sigmoid_cross_entropy_with_logits(
    logits=D_logit_fake, labels=tf.zeros_like(D_logit_fake)))
D_loss = D_loss_real + D_loss_fake
G_loss = tf.reduce_mean(tf.nn.sigmoid_cross_entropy_with_logits(
    logits=D_logit_fake, labels=tf.ones_like(D_logit_fake)))
```
# 五. GAN实现MNIST

## (一) 导入包
```python
import tensorflow as tf
import numpy as np
import matplotlib.pyplot as plt
import matplotlib.gridspec as gridspec
import os
from tensorflow.examples.tutorials.mnist import input_data
```
## (二) tensorflow设置
> 避免出现变量已经被使用，无法allow的情况
```python
tf.reset_default_graph()
```
## (三) 读取数据
```python
mnist = input_data.read_data_sets('MNIST_sets', one_hot=True)
```
## (四) 设置网络参数
```python
batch_size=128
learning_rate = 0.001
```
## (五) 参数——权重和偏置初始化
```python
def weight_var(shape, name):
    return tf.get_variable(name=name, shape=shape, initializer=tf.contrib.layers.xavier_initializer())

def bias_var(shape, name):
    return tf.get_variable(name=name, shape=shape, initializer=tf.constant_initializer(0))
```
## (六) 判别网络和生成网络参数设置
### 1.判别网络
```python
d_input=784
D=tf.placeholder(tf.float32,[None,d_input], name='D')

D_W1 = weight_var([784, 128], 'D_W1')
D_b1 = bias_var([128], 'D_b1')

D_W2 = weight_var([128, 1], 'D_W2')
D_b2 = bias_var([1], 'D_b2')

theta_D = [D_W1, D_W2, D_b1, D_b2]
```
### 2.生成网络
```python
g_input=100
G=tf.placeholder(tf.float32,[None,g_input], name='G')

G_W1 = weight_var([100, 128], 'G_W1')
G_b1 = bias_var([128], 'G_B1')

G_W2 = weight_var([128, 784], 'G_W2')
G_b2 = bias_var([784], 'G_B2')

theta_G = [G_W1, G_W2, G_b1, G_b2]
```
## (七) 网络结构
### 判别网络结构
> 判别网络设置 
> 输入层——784个结点；隐层（全连接层）——128个结点，激活函数relu；输出层——1个结点，激活函数sigmoid
```python
def discriminator(D):
    
    ######### input: 784 #########
    ######### 1 fc layer 128#########
    D_h1 = tf.nn.relu(tf.matmul(D, D_W1) + D_b1)
    
    ######### output layer 1 #########
    D_logit = tf.matmul(D_h1, D_W2) + D_b2
    
    ######### sigmoid #########
    D_prob = tf.nn.sigmoid(D_logit)
    return D_prob, D_logit
```
### 生成网络结构
> 生成网络设置 
> 输入层——100个结点；隐层（全连接层）——128个结点，激活函数relu；输出层——784个结点，激活函数sigmoid
```python
def generator(G):
    
    ######### input: 100 #########
    ######### 1 fc layer 128 #########
    G_h1 = tf.nn.relu(tf.matmul(G, G_W1) + G_b1)
    
    ######### output layer 784 #########
    G_log_prob = tf.matmul(G_h1, G_W2) + G_b2
    
    ######### sigmoid #########
    G_prob = tf.nn.sigmoid(G_log_prob)
    return G_prob
```
## (八) loss设定
```python
G_sample = generator(G)
D_real, D_logit_real = discriminator(D)
D_fake, D_logit_fake = discriminator(G_sample)

D_loss_real = tf.reduce_mean(tf.nn.sigmoid_cross_entropy_with_logits(
    logits=D_logit_real, labels=tf.ones_like(D_logit_real)))
D_loss_fake = tf.reduce_mean(tf.nn.sigmoid_cross_entropy_with_logits(
    logits=D_logit_fake, labels=tf.zeros_like(D_logit_fake)))
D_loss = D_loss_real + D_loss_fake

G_loss = tf.reduce_mean(tf.nn.sigmoid_cross_entropy_with_logits(
    logits=D_logit_fake, labels=tf.ones_like(D_logit_fake)))
```
## (九) 优化
```python
D_optimizer = tf.train.AdamOptimizer(learning_rate).minimize(D_loss, var_list=theta_D)
G_optimizer = tf.train.AdamOptimizer(learning_rate).minimize(G_loss, var_list=theta_G)
```

## (十) 生成器的初始输入
```python
def sample_G(m, n):
    return np.random.uniform(-1., 1., size=[m, n])
```
## (十一) 生成器学习结果存储路径
```python
if not os.path.exists('out/'):
    os.makedirs('out/')
```
## (十二) 生成器学习结果展示
```python
def plot(samples):
    fig = plt.figure(figsize=(4, 4))
    gs = gridspec.GridSpec(4, 4)
    gs.update(wspace=0.05, hspace=0.05)

    for i, sample in enumerate(samples):  # [i,samples[i]] imax=16
        ax = plt.subplot(gs[i])
        plt.axis('off')
        ax.set_xticklabels([])
        ax.set_aspect('equal')
        plt.imshow(sample.reshape(28, 28), cmap='Greys_r')

    return fig
```
## (十三) 训练
```python
init=tf.global_variables_initializer()
with tf.Session() as sess:
    sess.run(init)
    i = 0
    for it in range(1000000):
        
        ########## plot generator result ##########
        if it % 1000 == 0:
            samples = sess.run(G_sample, feed_dict={G: sample_G(16, g_input)})  # 16*100
            fig = plot(samples)
            plt.savefig('out/{}.png'.format(str(i).zfill(3)), bbox_inches='tight')
            i += 1
            plt.close(fig)
        
        ########## train sample ##########
        batch_x, _ = mnist.train.next_batch(batch_size)
        
        ########## D loss ##########
        _, D_loss_curr = sess.run([D_optimizer, D_loss], feed_dict={
                              D: batch_x, G: sample_G(batch_size, g_input)})
        
        ########## G loss ##########
        _, G_loss_curr = sess.run([G_optimizer, G_loss], feed_dict={
                              G: sample_G(batch_size, g_input)})
        
        ########## print loss ##########
        if it % 1000 == 0:
            print('Iter: {}'.format(it))
            print('D loss: {:.4}'.format(D_loss_curr))
            print('G_loss: {:.4}'.format(G_loss_curr))
            print()
```
## (十四) 结果
> 生成结果如下：取前12张

![Image text](https://github.com/ShaoQiBNU/Generative_Adversarial_Nets/blob/master/images/001.png)
![Image text](https://github.com/ShaoQiBNU/Generative_Adversarial_Nets/blob/master/images/002.png)
![Image text](https://github.com/ShaoQiBNU/Generative_Adversarial_Nets/blob/master/images/003.png)
![Image text](https://github.com/ShaoQiBNU/Generative_Adversarial_Nets/blob/master/images/004.png)
![Image text](https://github.com/ShaoQiBNU/Generative_Adversarial_Nets/blob/master/images/005.png)
![Image text](https://github.com/ShaoQiBNU/Generative_Adversarial_Nets/blob/master/images/006.png)
![Image text](https://github.com/ShaoQiBNU/Generative_Adversarial_Nets/blob/master/images/007.png)
![Image text](https://github.com/ShaoQiBNU/Generative_Adversarial_Nets/blob/master/images/008.png)
![Image text](https://github.com/ShaoQiBNU/Generative_Adversarial_Nets/blob/master/images/009.png)
![Image text](https://github.com/ShaoQiBNU/Generative_Adversarial_Nets/blob/master/images/010.png)
![Image text](https://github.com/ShaoQiBNU/Generative_Adversarial_Nets/blob/master/images/011.png)
![Image text](https://github.com/ShaoQiBNU/Generative_Adversarial_Nets/blob/master/images/012.png)
