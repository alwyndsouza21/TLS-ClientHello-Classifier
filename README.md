# TLS-ClientHello-Classifier
What it is in short->" A TLS-ClientHello packet classifier, where the features are enriched using GraphSAGE, before classification using a quite simple two layered neural network.", in case of any doubts feel free to reach out to me at [alwyn14120@gmail.com](mailot:alwyn14120@gmail.com).

Hi Friend, hope you are having a great day, i am happy that you have stopped by to read this!.

##Requirements: 

<b> These are some of the applications that you will need to create your very own dataset:), a good dataset is essential for great results!. </b>
1. <b>Wireshark</b>: Used to capture packets, while opening links(create a script to do this), stop the capture once all the links( normal) are captured. and save the .pcap file.
2. <b>Netcap</b>: Used to filter the .pcap files, I have used it to filter out TLS-ClientHello packets and save the result in a .json format. the data can be further transformed and processed in the script. I suggest you to visit the [NETCAP](https://netcap.io) website.
3. <b>VScode</b>: It is a widely used code editor, which is very convinient and practical, now this is much of a personal choice. Feel free to use any code editor of your choice!.
4. <b>Tshark</b>: This is the command line implementatio of Wireshark, it comes with Wiresharck usually but do check and install it if not already.

##Methodology:

There are three main modules, if you have the data ready. the first part is processing the data to remove features that donot correlate to the expected results. While i was analyzing dataset created by me, i found out that <b>Message_length</b> was a distinctive feature and so were some of the other features. To make the data cleaner with less noise, some new features are derived/egineered. the second and the most crucial part is that of creation of graph edges between the features. I have tried couple of ways of doing these, cosine similarity has been used for many such applications, because it considers the direction of vectors( feature vectors ), rather than their magnitude. Top-k similar nodes are used to create edges( in my experiments k value of 5 to 10, was found to be optimal). The last part which i consider important is classification, GraphSAGE learns to generate embeddings( features ), that provide a much generalize classes because of sampling and aggregation. The smaller the model, lesser the time it takes for inference. This is exactly what we need in a fast paced environment where there are millions of packets going back and forth at lightning speeds. I recommend you to read the [GraphSAGE paper](https://arxiv.org/abs/1706.02216).

Take a look at a simple block diagram that provides a abstract view of the complete project pipeline right from data generation to inference 👇🏼.

![Block Diagram](https://www.google.com/url?sa=i&url=https%3A%2F%2Fwww.hotstar.com%2Fin%2Fshows%2Fmr-bean%2F1971308273&psig=AOvVaw1QfPsxWQjMeP33F8J5VenL&ust=1754463139548000&source=images&cd=vfe&opi=89978449&ved=0CBUQjRxqFwoTCLCCvtOK844DFQAAAAAdAAAAABAE)

Feel free to tweak the code and boost the robustness of the approach!

- Alwyn D Souza



