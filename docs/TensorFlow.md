
#	TensorFlow



https://www.tensorflow.org/






```
import tensorflow as tf

model = tf.keras.models.Sequential()
```

Why "Sequential"?



```
#	model.add(tf.keras.layers.Dense(256, input_shape=x_train.shape, activation='sigmoid'))
#	model.add(tf.keras.layers.Dense(256, input_shape=(None, 455, 30), activation='sigmoid'))
#    ValueError: Input 0 of layer "sequential" is incompatible with the layer: expected shape=(None, 455, 30), found shape=(None, 30)

model.add(tf.keras.layers.Dense(256, activation='sigmoid'))
model.add(tf.keras.layers.Dense(256, activation='sigmoid'))
model.add(tf.keras.layers.Dense(1, activation='sigmoid'))
```

Is passing input_shape to the first layer better?

Why does shape from previous demo fail?

How many layers? Why?

Why 256?

Why sigmoid?

What are alternatives?

Why Dense?

What are alternatives?

Why is the last layer 1? Does it depend on the activation function?



```
model.compile(optimizer='adam', loss='binary_crossentropy', metrics=['accuracy'])
```

Why adam?

What are alternatives?

What are the differences?

Why binary_crossentropy?

What are alternatives?

What other options exist?







```

model.fit(x_train, y_train, batch_size=64, epochs=100)

```





```

model.evaluate(x_test, y_test)

```







##	Found Questions and Answers (not sure of the credibility)



What should batch size be TensorFlow?

Default value = 32

But we are limited to using the batch sizes with the power of 2 starting from 16 until 1024. This is because the batch size needs to fit the memory requirements of the GPU and the architecture of the CPU. So, the acceptable values for the batch size are 16, 32, 64, 128, 256, 512 and 1024!
model.fit(x_train, y_train, epochs=100)





What is the best number of neurons in a neural network?

The number of hidden neurons should be between the size of the input layer and the size of the output layer. 
The number of hidden neurons should be 2/3 the size of the input layer, plus the size of the output layer. 
The number of hidden neurons should be less than twice the size of the input layer.

That last statement doesn't seem to be necessary given the first 2 statements.



Followup: What is a hidden neuron?

Followup: What is a hidden layer?

A hidden layer in the context of artificial neural networks refers to a layer of neurons that is neither the input nor the output layer. 
Hidden layers are what make neural networks "deep" and enable them to learn complex data representations. 
They are the computational workhorse of deep learning models, allowing neural networks to approximate functions and capture patterns from input data.

So, not the first nor the last layer? Or just not the last layer?






