---
title: BraiNet - Password Less Authentication
subtitle: Using brain waves and machine learning for password-less authentication.
summary: Using brain waves and machine learning for password-less authentication.
date: 2017-08-06
math: true
diagram: true
markup: mmark
tags:
- Android 
- Python
categories:
- Machine-Learning
image:
  placement: 3
---

## Abstract
The password-based authentication methods have proved to be insecure because of attacks such as Phishing attacks and keystroke based attacks. This is starting to make the developers incline towards biometric-based authentication such as Fingerprint scanning and Retina scanning to make their applications more secure. But even these methods are not 100% foolproof. Many researchers have investigated the characteristics of the brain waves i.e. the electroencephalogram (EEG) signals and have observed that each person has a unique set of brain signals and these can be effectively used for authentication.

This project aims at developing a mobile application which authenticates the user on the basis of brain waves recorded using [Neurosky Mindwave Headset](http://developer.neurosky.com/docs/doku.php?id=mindwave). This application provides the functionality to store the brainwave data in database and query it. Users can also visualize their brainwave data with the help of a graph. The brainwave recognition has been implemented using **Dynamic Time Warping** as well as **machine learning algorithms like Naive Bayes, Stochastic Gradient Descent**.

## Keywords 
Brain sensor, Brain waves, Passwordless authentication, Fog Server

## Introduction
Authentication has always been very important factor in technology. Password based authentication schemes are quite common and have been there for quite some time. However, after numerous accounts of hacking attempts that were successful in bypassing the password based authentication methods, a need was felt to come up with a scheme that could not be compromised that easily. Here biometric based authentication came in handy. Since our biometric records such as fingerprints, retinas, brain signals are unique for each one of us, trying to impersonate someone using that person's biometric features is extremely difficult, if not impossible. Over the last few years, fingerprint based authentication has become prevalent. Retina based authentication is also quite popular in extremely high confidential and critical environments such as military. 

However, there is one more biometric feature that hasn't been fully tapped in yet; Our brainwaves. each person's brain wave signal is unique to that person only and is way too complex to be tried to be imitated which makes this a great authentication scheme. Sensor devices to record brain waves and stream them to devices such phones are available easily, which makes the implementation of this scheme, feasible.  


## Application Workflow

After the application is installed, the user has to register himself into the system if he is a first time user. On registration, an unique user ID will be generated and displayed to the user. 

<img src="/assets/images/home.png" width="200">

**Fig: Home screen**

<img src="/assets/images/register.png" width="200">

**Fig: Register screen**


The user can log in using this user ID. Clicking on login, user will be taken to a screen where he can connect to the Brain Sensor Device and Record his brain wave.  

<img src="/assets/images/login.png" width="200">

**Fig: Login screen**


<img src="/assets/images/device.png" width="200">

**Fig: Device Selection screen** 


<img src="/assets/images/record.png" width="200">

**Fig: Record screen** 

Recording can be done in two ways, until user hits the stop button or a timed record which will go on for 2 minutes. After recording, the system will authenticate the user by sending the data to the server where it will be analyzed.


<img src="/assets/images/Selection_015.png" width="200">

**Fig: Record screen** 


If the user is authenticated, he will be directed to a search page where user can access his own brain data by session. If the user is an admin user, he will have permission to access any users data and if he is not, he will only able to access his own data. 

<img src="/assets/images/search.png" width="200">

**Fig: Search screen** 


The data will be displayed to the user in the form of a plot.

<img src="/assets/images/search_result.png" width="200">

**Fig: Search Result screen** 


## Project Set up

![Project Setup](/assets/images/project-setup.png)
Fig: Project Setup

The Project set up primarily comprises of 4 components:

**1. Brain Sensor Device**
Neurosky Mindwave Mobile device is used for the brain sensor. This device streams data at a sampling rate of 512 Hz. It can be connected to mobile devices via Bluetooth.

**2.  Client Application**
This part is the android application that resides on the mobile phone. It collects data from the sensor device and sends the data to the web server for authentication and storage.

**3. Web Server**
The Mobile device-Server connection has been done via HTTP protocol. The Web server has been set up using Python-Flask framework.

**4. Database server**
MYSQL server is used for database. The database interacts with the our application via web server.

## Data Collection

The data collection is done by connecting the Neurosky Mindwave device to the mobile application via Bluetooth. Once the subject wears the sensor, the connection is established between the mobile app and the sensor device, the data is streamed to the application. The data collected by the application from the sensor is in raw form. The period of time for which the data is collected can be set in two ways:
1. One option is to start the process by pressing of Start button and is stopped by pressing of Stop Button. 
2. Second method is the Timed Recording option. In this option, upon pressing the start button, the data collection is done for a predetermined period of time after which, it stops automatically. 
 
The data is then sent to the web server, which directs the data to the database server where it is finally stored. 

## Implementation Details (Machine Learning)

### Feature Extraction

The alpha signals of the brain waves are used since they represent the relaxed awareness of the human brain. These waves have a frequency range of 8 to 13 Hz. After user logs into the application with his user ID and records its brain signal, the available brain signals for the same user are retrieved from the database. Both of these fetched and recorded brain signals are then divided into buckets of 512 data values and Fast Fourier Transform is applied to each bucket to get the set of 6 feature vectors. All the feature vectors of fetched signals (training data) are labeled as 1 and those of recorded signals (test data) are labeled as 0.

### Algorithm

The main idea in this authentication scheme is that a decision boundary is tried to be found between the feature vectors of fetched and recorded signals of the user. 
1. Both the training and test feature vectors obtained from feature extraction process are used collectively as the training data and fed to the model. 
2. The model then generates a decision boundary the best it can. 
3. The same training data is used as the test data and the confusion matrix is calculated.
 
**Why ?**

The main aim here is to check to how much extent the fetched and recorded data is separable by the decision boundary. Separability of data indicates the extent of the accurate decision boundary. 

    If confusionMat[0][0] > confusionMat[0][1] and confusionMat[1][1] > confusionMat[1][0]
    {
      authenticated = False
    }
    Else {
      authenticated = True
    }


4. The authentication decision relies on the confusion matrix. 
5. The interpretation of the Algorithm 1 is as follows: 

- If the decision boundary is correctly classifying both the samples, this means that the two samples are significantly different than each other. Hence, the user is not authenticated. 

- Whereas if it is incorrectly classifying samples, it represents that the two signals are similar and thus the user is authenticated.

## Results 

### Machine Learning Approach

Naive Bayes implemented for authentication achieved **54% of accuracy**.  It was observed that signals with smaller duration result in poor classifier performance. Thus, the model was tested on longer duration signals which significantly enhanced the performance.  
It was also observed that even though 2 brainwave signals are of the same person, they can still significantly differ from each other. 


![Project Setup](/assets/images/graph1.png)

Fig: Brain wave graph of subject A

![Project Setup](/assets/images/graph2.png)

Fig: Another Brain wave graph of subject A. Notice the difference from the previous graph.


## Challenges

For machine learning implementation, various classifiers such as Multilayer Perceptron(MLP), Support Vector Machines(SVM) and Stochastic Gradient Descent(SGD) were experimented with. But the results obtained from these classifiers were not good enough. 37%, 60% and 45% accuracies using the models MLP, SVM and SGD were obtained respectively. Even though SVM had higher accuracy, it authenticated the subject most of the times, signifying that the model was not performing correctly. Naive Bayes and Stochastic Gradient Descent performed better.

## Future Work

The results we got from our experiments did show that the machine learning approach is the way to go for trying to recognize the brain wave for authentication. However, the results were not totally unambiguous. There were scenarios where the classifier gets them wrong even though they are for the same subject. But on closely looking at the plotted graphs for those cases, we can see that even though the subject is same, the brainwaves are not that similar, meaning the results are highly dependent on the quality collection, which forms the future work for this project

## Conclusion
From this project, we observed that brainwave signals, being highly unique to each person , are a very good feature for being used in authentication schemes. We believe that as the wearable sensors become ubiquitous, the use of brainwave based authentication will becomes more and more feasible and prevalent and provide a way for us to effectively secure our data.

P.S - Here's the link to the [Github Repo](https://github.com/peps0791/BraiNet).
