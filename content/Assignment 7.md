Title: Classify Insects using deep learning model
Category: Deep Learning
Date: 2020-11-14 12:01
Modified: 2020-11-14 16:30
Tags: Dashboard, Visualization, Streamlit
Slug: deeplearning
Authors: Presnie Lu
Summary: Using a deep learning model to classify beetles, cockroaches and dragonflies

**This article will use a deep learning model: Multilayer Perceptrons, to classify beetkes, cockroaches and dragonflies images.**

# Dataset

This blog uses this [dataset](https://www.dropbox.com/s/fn73sj2e6c9rhf6/insects.zip?dl=0) and these images were splitted into training and test set. The goal is to predict the label of test set with high accuracy.

# Multilayer Perceptron
This blog will will use a multilayer perceptron model to make predictions. A multilayer perceptron (MLP) is a class of feedforward artificial neural network (ANN). Usually, an MLP have at least three layers of nodes: an input layer, a hidden layer and an output layer. Except for the input nodes, each node is a neuron that uses a nonlinear activation function like all other neural networks. MLP utilizes a supervised learning technique called backpropagation for training set. Therefore, the basic function of nonlinear separable patterns is carried out by adding a new layer called the hidden layer. Each input from the input layer is weighted and propagated through each node in hidden layer using an activation function to produce an output at the output layer.   
    
The training process in the MLP is consisted of forward and backward phases. In the forward phase, the error is calculated by minimizing any loss function. This error is used by the back propagation learning algorithm to adjust the weights. The output from each node is recalculated by using updated weights. Each time this process is repeated, the error is reduced.   
    
In general, MLPs are suitable for classification prediction problems where inputs are assigned a class or label. They are very flexible and can be used generally to learn a mapping from inputs to outputs. In terms of how do we use such model for image classification problem, I will the following diagram to explain the process.

![MLP](/img/MLP.png "Illustration of MLP")    

The diagram is a fairly simple example of image classification using MLP from the research paper "Pattern Recognition of Thermal Images for Monitoring of Breathing Function". It shows the process of classifying images with 9 features and uses an MLP with 1 hideden layer. In real life, as we know, images can usually be represented with pixels, which in this graph, is just the "feature". Therefore, a general process can be summarized as the following:  
    
   1. Image attributes are extracted from the image(pixels)
   2. Training image attributes are fed to the multilayer perceptron, adjusting weights through different back propagation. During this process, you can add as many hidden layers with any activation function as you want (notice that too complicated model usually has higher computational cost and has overfitting issue. )
   3. Final output layer generates outcome label or probabilities of belongs to each category
   
# Coding

This part will implement the MLP on classifying the insects images.  

## Preprocessing the data
The first step is to get the directory of all data:

    :::python 
    #directory
    dir_train = []
    dir_test = []
    for i in ['beetles', 'cockroach', 'dragonflies']:
        train_dir = r'insects/train/' + i
        test_dir = r'insects/test/' + i
        for fname in os.listdir(r'./' + train_dir):
            dir_train.append(train_dir+'/'+fname)
        for fname in os.listdir(r'./' + test_dir):
            dir_test.append(test_dir + '/' +fname)
   
Then, I used built two functions to generate our Xs and ys. Notice that I used Pillow library to convert an input jpeg to a 100x100 sized grey scale image array for processing. This step is used to make sure the size of images are consistent.   
    
    :::python
    #figure size
    def generate_X(path):
        data = []
        #resize 
        size = (100,100)
        for i in path:
            image = Image.open(i).convert('L')
            m_min_d = min(image.size)
            image = image.crop((0, 0, m_min_d, m_min_d))
            #scale data
            image.thumbnail(size, PIL.Image.ANTIALIAS)
            data.append(np.asarray(image))
        return np.asarray(data)
        
    def generate_Y(path):
        data = []
        for i in path:
            if re.match(r'.*beetles.*', i):
                data.append(0)
            elif re.match(r'.*cockroach.*', i):
                data.append(1)
            elif re.match(r'.*dragonflies.*', i):
                data.append(2)
        return np.asarray(data)
        
    random.seed(0)
    #training set
    random.shuffle(dir_train)
    random.shuffle(dir_test)
    X_train = generate_X(dir_train)
    y_train = generate_Y(dir_train)
    X_test = generate_X(dir_test)
    y_test = generate_Y(dir_test)

One additional step is required, which is scaling the images values between 0 and 1: 

    :::python
    # Scaling the images to values between 0 and 1
    X_train = X_train / 255.0
    X_test = X_test / 255.0

## Model Building 

For this problem, I set up four layers in total. The first step is to flatten the image data into a single array. The other three layers are dense layers. In terms of activation function, I tried sigmoid and relu as well as softmax using grid hyperparameter tuning. The following showed their actual functions:   
![Sigmoid](/img/Sigmoid.png "Sigmoid Activation Function")   
![ReLU](/img/ReLu.png "ReLU Activation Function")  
In addition, I also added the choice of the optimizer as stochastic gradient descent (SGD) or Adam for the compiler. The actual code looks like this:  

    :::python
    #hyperparameter tuning 
    def cv_parameter_tuning(X,y,activation1,activation2, optimization):
    cv_acc = {}
    kf = KFold(n_splits=3,random_state=0)
    for a1 in activation1:
        for a2 in activation2:
            for opt in optimization:
                fold_result = []
                for train, val in kf.split(X):
                    x_train1,y_train1 =  X[train,:,:], y[train]
                    x_val, y_val = X[val,:,:], y[val]
                    random.seed(1)
                    model = keras.Sequential([
                                keras.layers.Flatten(input_shape=(100, 100)),
                                keras.layers.Dense(128, activation=a1),
                                keras.layers.Dense(16, activation=a1),
                                keras.layers.Dense(3, activation=a2)
                                ])
            
                    model.compile(optimizer=opt,
                                   loss='sparse_categorical_crossentropy',
                                   metrics=['accuracy'])

                    model.fit(x_train1, y_train1,epochs=100, verbose=0)
                    loss, acc = model.evaluate(x_val, y_val, verbose=0)
                    
                    fold_result.append(acc)
                    
                combo = "a1:"+str(a1) + " " + "a2:" + str(a2) + " " + "opt:" + str(opt)
                #take the average and std of all fold's accuracy, append it to the list
                cv_acc[combo] = np.mean(fold_result)
                
    return cv_acc
    
    grid = cv_parameter_tuning(X_train,y_train,[ 'relu', 'sigmoid'], [ 'softmax'], ["SGD","Adam"])
    
From the grid search cross validation process, I chose the first two combinations with highest accuracy level, which are 1.'a1:sigmoid a2:softmax opt:Adam' and 2. 'a1:sigmoid a2:softmax opt:SGD'       

    :::python
    # model building
    
    #model 1
    random.seed(1)
    model = keras.Sequential([
        keras.layers.Flatten(input_shape=(100, 100)),
            keras.layers.Dense(128, activation=tf.nn.sigmoid),
            keras.layers.Dense(16, activation=tf.nn.sigmoid),
        keras.layers.Dense(3, activation=tf.nn.softmax)
    ])

    model.compile(optimizer="Adam",
              loss='sparse_categorical_crossentropy',
              metrics=['accuracy'])

    model_hist = model.fit(X_train, y_train,epochs=100)
    test_loss, test_acc = model.evaluate(X_test, y_test)
    print('Test accuracy:', test_acc)
    
    #model 2
    random.seed(1)
    model2 = keras.Sequential([
        keras.layers.Flatten(input_shape=(100, 100)),
            keras.layers.Dense(128, activation=tf.nn.sigmoid),
            keras.layers.Dense(16, activation=tf.nn.sigmoid),
        keras.layers.Dense(3, activation=tf.nn.softmax)
    ])

    model2.compile(optimizer="SGD",
              loss='sparse_categorical_crossentropy',
              metrics=['accuracy'])

    model_hist2 = model2.fit(X_train, y_train,epochs=100)
    test_loss2, test_acc2 = model2.evaluate(X_test, y_test)
    print('Test accuracy:', test_acc2)
    

## Model Evaluation

From the final outcome, model 1 ('a1:sigmoid a2:softmax opt:Adam') performs much better in test set with a loss of 0.59 and accuracy level of 0.9.  
![Result](/img/Acc_m.png "Final model accuracy")   

The following plot shows the prediction accuracy of training set as epoch size increases. As we can see that as the epoch number increases, the accuracy level of training data is getting higher (0.92) and it also getting more stable than before. From this we can see the dataset is also not overfitting as the training and test set has similar accuracy level.  
![Accuracy](/img/prediction_train.png "Training accuracy")   

Then, the next step is to see the actual predicition for our dataset. Because the model will render probabilities of each category for images, I just chose the category with the highest probability as the final category. 

    :::python
    #prediction
    predictions = model.predict(X_test)
    def display_images(images, labels):
        plt.figure(figsize=(10,10))
        grid_size = min(25, len(images))
        for i in range(grid_size):
                plt.subplot(5, 5, i+1)
                plt.xticks([])
                plt.yticks([])
                plt.grid(False)
                plt.imshow(images[i], cmap=plt.cm.binary)
                plt.xlabel(class_names[labels[i]])
    class_names = ['beetles', 'cockroach', 'dragonflies']
    display_images(X_test,np.argmax(predictions, axis = 1))

The following showed the 25 images (order changed by shuffling; the image is grey scaled because of our preprocessing):    
![Prediction](/img/insects_pred.png "Prediction")   
With the actual classification (1. 'beetles'; 2. 'cockroach'; 3. 'dragonflies')  
![actual](/img/actual_c.png "Actual")  
We can see that most of the images got correctly classified except ones with very obscure representation of insects. (For intance, some only shows parts of the insects and human can even hardly identify which insects they belong to.) Therefore, we can say that our model is doing generally good. 

