
![mockingbird](https://user-images.githubusercontent.com/12797292/131216767-6eb251d6-14fc-4951-8324-2722f0cd4c63.jpg)




## Features
🌍 **Chinese** supported mandarin and tested with multiple datasets: aidatatang_200zh, magicdata, aishell3, data_aishell, and etc.

🤩 **PyTorch** worked for pytorch, tested in version of 1.9.0(latest in August 2021), with GPU Tesla T4 and GTX 2060

🌍 **Windows + Linux** run in both Windows OS and linux OS (even in M1 MACOS)

🤩 **Easy & Awesome** effect with only newly-trained synthesizer, by reusing the pretrained encoder/vocoder

🌍 **Webserver Ready** to serve your result with remote calling

## Quick Start

### 1. Install Requirements
#### 1.1 General Setup
> Follow the original repo to test if you got all environment ready.
**Python 3.7 or higher ** is needed to run the toolbox.

* Install [PyTorch](https://pytorch.org/get-started/locally/).
> If you get an `ERROR: Could not find a version that satisfies the requirement torch==1.9.0+cu102 (from versions: 0.1.2, 0.1.2.post1, 0.1.2.post2 )` This error is probably due to a low version of python, try using 3.9 and it will install successfully
* Install [ffmpeg](https://ffmpeg.org/download.html#get-packages).
* Run `pip install -r requirements.txt` to install the remaining necessary packages.
> The recommended environment here is `Repo Tag 0.0.1` `Pytorch1.9.0 with Torchvision0.10.0 and cudatoolkit10.2` `requirements.txt` `webrtcvad-wheels` because `requirements. txt` was exported a few months ago, so it doesn't work with newer versions
* Install webrtcvad `pip install webrtcvad-wheels`(If you need)

or
- install dependencies with `conda` or `mamba`

  ```conda env create -n env_name -f env.yml```

  ```mamba env create -n env_name -f env.yml```

  will create a virtual environment where necessary dependencies are installed. Switch to the new environment by `conda activate env_name` and enjoy it.
  > env.yml only includes the necessary dependencies to run the project，temporarily without monotonic-align. You can check the official website to install the GPU version of pytorch.

#### 1.2 Setup with a M1 Mac
> The following steps are a workaround to directly use the original `demo_toolbox.py`without the changing of codes.
>
  >  Since the major issue comes with the PyQt5 packages used in `demo_toolbox.py` not compatible with M1 chips, were one to attempt on training models with the M1 chip, either that person can forgo `demo_toolbox.py`, or one can try the `web.py` in the project.

##### 1.2.1 Install `PyQt5`, with [ref](https://stackoverflow.com/a/68038451/20455983) here.
  * Create and open a Rosetta Terminal
  * Use system Python to create a virtual environment for the project
    ```
    /usr/bin/python3 -m venv /PathToMockingBird/venv
    source /PathToMockingBird/venv/bin/activate
    ```
  * Upgrade pip and install `PyQt5`
    ```
    pip install --upgrade pip
    pip install pyqt5
    ```
##### 1.2.2 Install `pyworld` and `ctc-segmentation`

> Both packages seem to be unique to this project and are not seen in the original [Real-Time Voice Cloning] project. When installing with `pip install`, both packages lack wheels so the program tries to directly compile from c code and could not find `Python.h`.

  * Install `pyworld`
      * `brew install python` `Python.h` can come with Python installed by brew
      * `export CPLUS_INCLUDE_PATH=/opt/homebrew/Frameworks/Python.framework/Headers` The filepath of brew-installed `Python.h` is unique to M1 MacOS and listed above. One needs to manually add the path to the environment variables.
      * `pip install pyworld` that should do.


  * Install`ctc-segmentation`
    > Same method does not apply to `ctc-segmentation`, and one needs to compile it from the source code on [github].
    * `git clone https://github.com/lumaku/ctc-segmentation.git`
    * `cd ctc-segmentation`
    * `source /PathToMockingBird/venv/bin/activate` If the virtual environment hasn't been deployed, activate it.
    * `cythonize -3 ctc_segmentation/ctc_segmentation_dyn.pyx`
    * `/usr/bin/arch -x86_64 python setup.py build` Build with x86 architecture.
    * `/usr/bin/arch -x86_64 python setup.py install --optimize=1 --skip-build`Install with x86 architecture.

##### 1.2.3 Other dependencies
  * `/usr/bin/arch -x86_64 pip install torch torchvision torchaudio` Pip installing `PyTorch` as an example, articulate that it's installed with x86 architecture
  * `pip install ffmpeg`  Install ffmpeg
  * `pip install -r requirements.txt` Install other requirements.

##### 1.2.4 Run the Inference Time (with Toolbox)
  > To run the project on x86 architecture. [ref]
  * `vim /PathToMockingBird/venv/bin/pythonM1` Create an executable file `pythonM1` to condition python interpreter at `/PathToMockingBird/venv/bin`.
  * Write in the following content:
    ```
    #!/usr/bin/env zsh
    mydir=${0:a:h}
    /usr/bin/arch -x86_64 $mydir/python "$@"
    ```
  * `chmod +x pythonM1` Set the file as executable.
  * If using PyCharm IDE, configure project interpreter to `pythonM1` if using command line python, run `/PathToMockingBird/venv/bin/pythonM1 demo_toolbox.py`


### 2. Prepare your models
> Note that we are using the pretrained encoder/vocoder but not synthesizer, since the original model is incompatible with the Chinese symbols. It means the demo_cli is not working at this moment, so additional synthesizer models are required.

You can either train your models or use existing ones:

#### 2.1 Train encoder with your dataset (Optional)

* Preprocess with the audios and the mel spectrograms:
`python encoder_preprocess.py <datasets_root>` Allowing parameter `--dataset {dataset}` to support the datasets you want to preprocess. Only the train set of these datasets will be used. Possible names: librispeech_other, voxceleb1, voxceleb2. Use comma to sperate multiple datasets.

* Train the encoder: `python encoder_train.py my_run <datasets_root>/SV2TTS/encoder`
> For training, the encoder uses visdom. You can disable it with `--no_visdom`, but it's nice to have. Run "visdom" in a separate CLI/process to start your visdom server.

#### 2.2 Train synthesizer with your dataset
* Download dataset and unzip: make sure you can access all .wav in folder
* Preprocess with the audios and the mel spectrograms:
`python pre.py <datasets_root>`
Allowing parameter `--dataset {dataset}` to support aidatatang_200zh, magicdata, aishell3, data_aishell, etc.If this parameter is not passed, the default dataset will be aidatatang_200zh.

* Train the synthesizer:
`python train.py --type=synth mandarin <datasets_root>/SV2TTS/synthesizer`

* Go to next step when you see attention line show and loss meet your need in training folder *synthesizer/saved_models/*.

#### 2.3 Use pretrained model of synthesizer
#### 2.4 Train vocoder (Optional)
> note: vocoder has little difference in effect, so you may not need to train a new one.
* Preprocess the data:
`python vocoder_preprocess.py <datasets_root> -m <synthesizer_model_path>`
> `<datasets_root>` replace with your dataset root，`<synthesizer_model_path>`replace with directory of your best trained models of sythensizer, e.g. *sythensizer\saved_mode\xxx*

* Train the wavernn vocoder:
`python vocoder_train.py mandarin <datasets_root>`

* Train the hifigan vocoder
`python vocoder_train.py mandarin <datasets_root> hifigan`

### 3. Launch
#### 3.1 Using the web server
You can then try to run:`python web.py` and open it in browser, default as `http://localhost:8080`

#### 3.2 Using the Toolbox
You can then try the toolbox:
`python demo_toolbox.py -d <datasets_root>`

#### 3.3 Using the command line
You can then try the command:
`python gen_voice.py <text_file.txt> your_wav_file.wav`
you may need to install cn2an by "pip install cn2an" for better digital number result.
