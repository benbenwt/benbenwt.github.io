- 示例
  代码见tensorflow官网或qq收藏
  读取文件夹数据:tf.keras.preprocessing.image_dataset_from_directory,分为训练集和验证集
  train_ds.class_names获取分类标签名
  for images, labels in train_ds.take(1):遍历
  打乱数据，创建缓冲，归一化，封装batch。归一化可以放在model外，也可以成为一层网络。
  AUTOTUNE = tf.data.experimental.AUTOTUNE
  train_ds = train_ds.cache().shuffle(1000).prefetch(buffer_size=AUTOTUNE)
  val_ds = val_ds.cache().prefetch(buffer_size=AUTOTUNE)
  normalization_layer = layers.experimental.preprocessing.Rescaling(1./255)
  normalized_ds = train_[ds.map](http://ds.map/)(lambda x, y: (normalization_layer(x), y))
  image_batch, labels_batch = next(iter(normalized_ds))
  创建model
   model = Sequential([#   layers.experimental.preprocessing.Rescaling(1./255, input_shape=(img_height, img_width, 3)),
  \#   layers.Conv2D(16, 3, padding='same', activation='relu'),
  \#   layers.MaxPooling2D(),
  \#   layers.Conv2D(32, 3, padding='same', activation='relu'),
  \#   layers.MaxPooling2D(),
  \#   layers.Conv2D(64, 3, padding='same', activation='relu'),
  \#   layers.MaxPooling2D(),
  \#   layers.Flatten(),
  \#   layers.Dense(128, activation='relu'),
  \#   layers.Dense(num_classes)# ])
  编译model
  model.compile(optimizer='adam',
  \# loss=tf.keras.losses.SparseCategoricalCrossentropy(from_logits=True),
  \# metrics=['accuracy'])
  展示网络结构
   model.summary()
  训练
  \## epochs=10
  \# history = [model.fit](http://model.fit/)(#   train_ds,#   validation_data=val_ds,#   epochs=epochs# )
  \#保存模型
   [model.save](http://model.save/)('my_model1.h5')
  \# model = tf.keras.models.load_model('my_model1.h5')
  \# model.summary()
  图像处理：
  data_augmentation = keras.Sequential(
    [
      layers.experimental.preprocessing.RandomFlip("horizontal",
  input_shape=(img_height,
                                                                img_width,
  3)),
      layers.experimental.preprocessing.RandomRotation(0.1),
      layers.experimental.preprocessing.RandomZoom(0.1),
    ]
  )
  augmented_images = data_augmentation(images)

  

  

  

  

  

- 预处理

  - 创建图片预处理及使用方法
    1使用keras.layers.experiment.processiong.RandomFlip创建
    2使用自定义函数创建，返回处理后数值。

    对于1可以作为model的网络的一层进行fit处理，也可以构建单独的Sequential进行处理。
    或不封装为Sequential，使用map修改图片。
    对于2只能使用map处理图片。

    使用map处理的结果，用next（iter）转为batch，即可访问。
    使用Sequential的直接for便利，同dataset。