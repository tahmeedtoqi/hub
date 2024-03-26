---
layout: hub_detail
background-class: hub-background
body-class: hub
title: WaveGlow
summary: WaveGlow model for generating speech from mel spectrograms (generated by Tacotron2)
category: researchers
image: nvidia_logo.png
author: NVIDIA
tags: [audio]
github-link: https://github.com/NVIDIA/DeepLearningExamples/tree/master/PyTorch/SpeechSynthesis/Tacotron2
github-id: NVIDIA/DeepLearningExamples
featured_image_1: waveglow_diagram.png
featured_image_2: no-image
accelerator: cuda
order: 10
demo-model-link: https://huggingface.co/spaces/pytorch/WaveGlow
---


### Model Description

The Tacotron 2 and WaveGlow model form a text-to-speech system that enables user to synthesise a natural sounding speech from raw transcripts without any additional prosody information. The Tacotron 2 model (also available via torch.hub) produces mel spectrograms from input text using encoder-decoder architecture. WaveGlow is a flow-based model that consumes the mel spectrograms to generate speech.

### Example

In the example below:
- pretrained Tacotron2 and Waveglow models are loaded from torch.hub
- Given a tensor representation of the input text ("Hello world, I missed you so much"), Tacotron2 generates a Mel spectrogram as shown on the illustration
- Waveglow generates sound given the mel spectrogram
- the output sound is saved in an 'audio.wav' file

To run the example you need some extra python packages installed.
These are needed for preprocessing the text and audio, as well as for display and input / output.
```bash
pip install numpy scipy librosa unidecode inflect librosa
apt-get update
apt-get install -y libsndfile1
```

Load the WaveGlow model pre-trained on [LJ Speech dataset](https://keithito.com/LJ-Speech-Dataset/)
```python
import torch
waveglow = torch.hub.load('NVIDIA/DeepLearningExamples:torchhub', 'nvidia_waveglow', model_math='fp32')
```

Prepare the WaveGlow model for inference
```python
waveglow = waveglow.remove_weightnorm(waveglow)
waveglow = waveglow.to('cuda')
waveglow.eval()
```

Load a pretrained Tacotron2 model
```python
tacotron2 = torch.hub.load('NVIDIA/DeepLearningExamples:torchhub', 'nvidia_tacotron2', model_math='fp32')
tacotron2 = tacotron2.to('cuda')
tacotron2.eval()
```

Now, let's make the model say:
```python
text = "hello world, I missed you so much"
```

Format the input using utility methods
```python
utils = torch.hub.load('NVIDIA/DeepLearningExamples:torchhub', 'nvidia_tts_utils')
sequences, lengths = utils.prepare_input_sequence([text])
```

Run the chained models
```python
with torch.no_grad():
    mel, _, _ = tacotron2.infer(sequences, lengths)
    audio = waveglow.infer(mel)
audio_numpy = audio[0].data.cpu().numpy()
rate = 22050
```

You can write it to a file and listen to it
```python
from scipy.io.wavfile import write
write("audio.wav", rate, audio_numpy)
```

Alternatively, play it right away in a notebook with IPython widgets
```python
from IPython.display import Audio
Audio(audio_numpy, rate=rate)
```

### Details
For detailed information on model input and output, training recipies, inference and performance visit: [github](https://github.com/NVIDIA/DeepLearningExamples/tree/master/PyTorch/SpeechSynthesis/Tacotron2) and/or [NGC](https://ngc.nvidia.com/catalog/resources/nvidia:tacotron_2_and_waveglow_for_pytorch)

### References

 - [Natural TTS Synthesis by Conditioning WaveNet on Mel Spectrogram Predictions](https://arxiv.org/abs/1712.05884)
 - [WaveGlow: A Flow-based Generative Network for Speech Synthesis](https://arxiv.org/abs/1811.00002)
 - [Tacotron2 and WaveGlow on NGC](https://ngc.nvidia.com/catalog/resources/nvidia:tacotron_2_and_waveglow_for_pytorch)
 - [Tacotron2 and Waveglow on github](https://github.com/NVIDIA/DeepLearningExamples/tree/master/PyTorch/SpeechSynthesis/Tacotron2)