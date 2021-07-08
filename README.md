
# Mini Speech Training with LSTM

This example shows how to train a 500 kB model that can recognize any of 8 keywords chosen by the user,
classify all other commands as an "unknown" keyword, and predict the chosen keywords from speech data.

You can retrain it to recognize any combination of words (2 or more) from this
list (all other words would be passed to "unknown" keyword set):

```
yes
no
up
down
left
right
on
off
stop
go
```

The scripts used in training the model have been sourced from the
[Simple Audio Recognition](https://www.tensorflow.org/tutorials/audio/simple_audio)
tutorial.

## Table of contents

-   [Overview](#overview)
-   [Training](#training)
-   [Trained Models](#trained-models)
-   [Model Architecture](#model-architecture)
-   [Dataset](#dataset)
-   [Preprocessing Speech Input](#preprocessing-speech-input)


## Overview

1.  Dataset: Mini Speech Commands, Version 2.
    ([Download Link](http://storage.googleapis.com/download.tensorflow.org/data/mini_speech_commands.zip),
    [Paper](https://arxiv.org/abs/1804.03209))
2.  Dataset Type: **Mini_Speech_Commands**
3.  Deep Learning Framework: **TensorFlow 2.5.0**
4.  Language: **Python 3.7**
5.  Model Size: **<20 kB**
6.  Model Category: **Multiclass Classification**

## Training

Train the model in the cloud using Google Colaboratory.

<table class="tfo-notebook-buttons" align="left">
  <td>
    <a target="_blank" href="https://colab.research.google.com/github/tensorflow/tensorflow/blob/master/tensorflow/lite/micro/examples/micro_speech/train/train_micro_speech_model.ipynb"><img src="https://www.tensorflow.org/images/colab_logo_32px.png" />Google Colaboratory</a>
  </td>


*Estimated Training Time: ~2 Minutes.*


## Trained Models

| Download Link        | [speech_commands.zip](https://storage.googleapis.com/download.tensorflow.org/models/tflite/micro/micro_speech_2020_04_13.zip)           |
| ------------- |-------------|

The `models` directory in the above zip file can be generated by following the
instructions in the [Training](#training) section above. It
includes the following 3 model files:

| Name           | Format       | Target Framework | Target Device             |
| :------------- | :----------- | :--------------- | :------------------------ |
| `micro_speech_lstm_model.pb`     | Frozen       | TensorFlow       | Large-Scale/Cloud/Servers |
:                : GraphDef     :                  :                           :
| `micro_speech_lstm_model.tflite` | Fully        | TensorFlow Lite  | Mobile Devices            |
: *(<500 kB)*     : Quantized*   :                  :                           :
:                : TFLite Model :                  :                           :
| `micro_speech_lstm_model.cc`     | C Source     | TensorFlow Lite  | Microcontrollers          |
:                : File         : for              :                           :
:                :              : Microcontrollers :                           :

**Fully quantized implies that the model is **strictly int8** quantized
**including** the input(s) and output(s).*
<!-- **Fully quantized implies that the model is **strictly int8** except the
input(s) and output(s) which remain float.* -->

## Model Architecture

This is a simple model comprising of a Unidirectional Sequence LSTM layer, a Reshape layer, a Fully Connected
Layer or a MatMul Layer (output: logits) and a Softmax layer
(output: probabilities) as shown below. Refer to the [`tiny_conv`](https://github.com/tensorflow/tensorflow/blob/master/tensorflow/examples/speech_commands/models.py#L673)
model architecture.

![model_architecture.png](../images/model_architecture.png)

*This image was derived from visualizing the 'model.tflite' file in
[Netron](https://github.com/lutzroeder/netron)*

This doesn't produce a highly accurate model, but it's designed to be used as
the first stage of a pipeline, running on a low-energy piece of hardware that
can always be on, and then wake higher-power chips when a possible utterance has
been found, so that more accurate analysis can be done. Additionally, the model
takes in preprocessed speech input as a result of which we can leverage a
simpler model for accurate results.

## Dataset

The Mini Speech Commands Dataset. ([Download Link](http://storage.googleapis.com/download.tensorflow.org/data/mini_speech_commands.zip),
[Paper](https://arxiv.org/abs/1804.03209)) consists of over 8,000 WAVE audio
files of people saying 8 different words. This data was collected by
Google and released under a CC BY license. You can help improve it by
contributing five minutes of your own voice. The archive is over 2GB, so this
part may take a while, but you should see progress logs, and once it's been
downloaded you won't need to do this again.

## Preprocessing Speech Input

In this section we discuss spectrograms, the preprocessed speech input to the
model. Here's an illustration of the process:

![spectrogram diagram](https://storage.googleapis.com/download.tensorflow.org/example_images/spectrogram_diagram.png)

The model doesn't take in raw audio sample data, instead it works with
spectrograms which are two dimensional arrays that are made up of slices of
frequency information, each taken from a different time window.

The recipe for creating the spectrogram data is that each frequency slice is
created by running an FFT across a 30ms section of the audio sample data. The
input samples are treated as being between -1 and +1 as real values (encoded as
-32,768 and 32,767 in 16-bit signed integer samples).

This results in an FFT with 256 entries. Every sequence of six entries is
averaged together, giving a total of 43 frequency buckets in the final slice.
The results are stored as unsigned eight-bit values, where 0 represents a real
number of zero, and 255 represents 127.5 as a real number.

Each adjacent frequency entry is stored in ascending memory order (frequency
bucket 0 at data[0], bucket 1 at data[1], etc). The window for the frequency
analysis is then moved forward by 20ms, and the process repeated, storing the
results in the next memory row (for example bucket 0 in this moved window would
be in data[43 + 0], etc). This process happens 49 times in total, producing a
single channel image that is 43 pixels wide, and 49 rows high.

In a complete application these spectrograms would be calculated at runtime from
microphone inputs, but the code for doing that is not yet included in this
sample code. The test uses spectrograms that have been pre-calculated from
one-second WAV files in the test dataset generated by running the following
commands:



