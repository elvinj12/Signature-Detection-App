# Signature-Detection-App
This is a Signature detection app to which a trained deep learning model is deployed 

Versions:
Tensorflow= 1.15
Protoc 3.4
I used Python 3.7.7. (Latest Anaconda is Python 3.8. This should work fine. However, another way to install python 3.7 I found is to use conda install -c anaconda python=3.7.7 after creating an environment in the steps below, but I think this would not be needed)
(Remaining libraries are automatically come with Anaconda, when it is installed)
 
First time setup:
-          Download and install Anaconda exe file from here: https://repo.anaconda.com/archive/Anaconda3-2020.07-Windows-x86_64.exe
-          Open Anaconda Prompt from Start (Windows)
-          Create a new anaconda environment by typing the following:
-          conda create -n ‘name of new env’
o   Then do: conda activate ‘name of new env’
-          Run this command to download needed packages : pip install TensorFlow==1.15 lxml pillow matplotlib jupyter contextlib2 cython tf_slim  
o   (only TensorFlow 1.x.x works so use 1.15)
-          If you have git installed, go to where you want to download the repository and run: git clone https://github.com/tensorflow/models.git
o   If you don’t have git go to the above link and download the zip and extract it
-          Next go here to download protoc 3.4 (only this version works properly)
o   Extract the zip and open the bin folder. Move protoc.exe into the ‘research’ folder of the cloned repository (models-master repository extracted in the previous step)
o   From the ‘research’ directory run this command to compile protoc: protoc object_detection/protos/*.proto --python_out=.
-          Go to the ‘research’ directory and run these two commands:
o   Python setup.py build
o   Python setup.py install
-          Also run pip install pycocotools
o   If this does not work, instead run this command which grabs the wheel file needed  pip install git+https://github.com/philferriere/cocoapi.git#egg=pycocotools^&subdirectory=PythonAPI
If it still doesn’t work use: pip install pycocotools-windows

This completes the installation procedure
Creation of the dataset
1) Using Labelimg
1.	Because we are using object detection to extract the required object from the image, we cannot simply feed the models with images. We also need to annotate (extract the bounding boxes) from the images. For this, we need to use Labelimg, which is a tool that can be downloaded from here.
2.	Extract the zip file and run the application. Now select the ‘Open directory’ on the left pane to select the directory where the images are saved. Make sure that the file format is set to ‘Pascal VOC’ and not ‘Yolo’ on the left pane. Create the bounding box around the signatures using the ‘Create RectBox’ option on the left pane with the help of the mouse. 
3.	Once the box is drawn a window will pop up, where we need to mention the class to which this belongs. Simply type ‘DLSignature’ and click OK. Then, using the ‘save’ option on the left pane, save the xml file which will be generated as output in a directory.
2) Converting XML to CSV
1.	Once all the all the xml files are ready, split them into two folders, depending on whether that image was to be used for training or validation, in a folder called ‘train’ and ‘validation’. These xml files will now have all the information needed, for generating the excel file having the annotations.
2.	To Create the excel file (annotations_train.csv), use the ‘Create dataset’ code. It must be given the locations to all the xml files for training appropriately. Do the same for the validation xml files. At the end of this step, 2 files will be ready ‘annotations_train.csv’ and ‘annotations_validation.csv’.
3) Generating the tf.record files
1.	Once the excel is ready, we are ready to generate the final dataset files. Now, we need to run the generate_tfrecord.py code. Place this code in the folder where the excel files are present. Along with this, there should be a folder named ‘images’ which has all the images for which the annotations were prepared (train+validation images together, no need to separate them).
2.	Run the code by entering the following in the anaconda prompt, by cd’ing into the directory where the excel files generated in the previous step are present.
3.	python generate_tfrecord.py --csv_input=annotations_train.csv  --output_path=train.record --image_dir=images/
4.	then do the same for validation as well
5.	python generate_tfrecord.py --csv_input=annotations_train.csv  --output_path=train.record --image_dir=images/
6.	At the end of these steps, 2 files will be ready train.record and validation.record. 
7.	Next we need to prepare a .txt file which has the names of the classes. Simple open notepad type in 
item: {
	id:  1
	name: 'DLSignature'
}
And save it as object-detection.pbtxt.
4)Preparing folders in the models-master/research/object_detection directory
1.	Now open the directory models-master/research/object_detection (Downloaded during installation). This is where the rest of the work is to be done.
2.	Create a folder called ‘images’. Create two folders train and validation inside this. Inside the train folder paste the train.record file and the object-detction.pbtxt. file Repeat the same of the validation folder with the validation.record file
3.	Next, create a folder called data in the object_detection directory. Place a copy of the object_detection.pbtxt file here too.
4.	Next we need to download the models from github. Select the  faster_rcnn_inception_resnet_v2_atrous_coco and download and extract it to the object_detection folder. 
5.	Next open samples/configs in the and select the corresponding config file for the model that you had just downloaded. Now we need to edit this. Make the following changes:
num_classes: 1
batch_size: 1
fine_tune_checkpoint: "faster_rcnn_inception_resnet_v2_atrous_coco_2018_01_28/model.ckpt"
input_path: "images/train/train.record"
label_map_path: "images/train/object-detection.pbtxt"
input_path: "images/validation/validation.record"
label_map_path: "images/validation/object-detection.pbtxt"

Also,
Add these 3 lines below the above batch_size argument
  batch_queue_capacity: 50
  num_batch_queue_threads: 8
  prefetch_queue_capacity: 10

And add these 2 lines below the label_map_path argument in both the train eval reader section
queue_capacity: 500
min_after_dequeue: 250
And add these 4 lines below the label_map_path argument in eval reader section
  queue_capacity: 500
  min_after_dequeue: 250
  shuffle: false
  num_readers: 1
6.	Save this file now and create a new folder called training in the object_detection folder. Paste the config file here and also a copy of the object_detection.pbtxt made earlier
5) Training
1.	Now, we are ready to train.
2.	Upload the entire models-master directory to google drive.
3.	Also, upload the ‘Model training Colab.ipynb’ code. Run the cells one by one by clicking the play button on each cell (till the cell that has the heading ‘Start training’ and training will begin. Remember to make changes in the code as per the directory hierarchy of you google drive. )
4.	Once training has been stopped, navigate to the ‘training’ directory on drive. Select the number of the latest checkpoint that was saved (based on the time it was saved) and enter the number in the ‘Model training Colab.ipynb’ cell, in the cell after the training cell and paste the number For eg if the number is 2755 then edit the argument and run the cell. 
--trained_checkpoint_prefix training/model.ckpt-2755 
5.	A folder called new_graph will be created, in the object detection directory.

