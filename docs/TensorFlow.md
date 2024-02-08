
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

model.fit(x_train, y_train, epochs=100)

model.evaluate(x_test, y_test)

```

