# TesseractTrainer
by Balthazar Rouberol - [Mor Consulting](http://morconsulting.com/), [rouberol.b@gmail.com](mailto:rouberol.b@gmail.com)

TesseractTrainer is a simple Python API, taking over the tedious process of manually
training Tesseract3, as described in the [wiki page](https://code.google.com/p/tesseract-ocr/wiki/TrainingTesseract3).

The longest part of the training process is checking the box file, generated by tesseract using a reference tif image,
as explained [here](https://code.google.com/p/tesseract-ocr/wiki/TrainingTesseract3#Make_Box_Files).
This file contains the coordinates of each character detected in the training tif. However, if Tesseract made
some mistakes, you have to manually correct the boxfile, allowing Tesseract to "learn" from its mistakes.

TesseractTrainer allows you to skip this part, by automatically generating a tif (and the associated boxfile) using a
text and a font that you specify, thus **guaranteeing the total accuracy of the box file**.

TesseractTrainer intends to provide both a python API and a bash command line tool.

## Dependencies

* a Unix/Linux system
* Tesseract3.x
* python 2.6 - 2.7
* PIL (Python Imaging Library). Note that PIL has not yet been ported to Python 3.
* ImageMagick

## Installation

You can install TesseractTrainer in your virtualenv via

```
pip install TesseractTrainer
```

or

```
easy_install TesseractTrainer
```

if you really (really) have to.


## Command line tool
```bash
usage: tesstrain [-h]
                  --tesseract-lang TESSERACT_LANG
                  --training-text TRAINING_TEXT
                  --font-path FONT_PATH
                  --font-name FONT_NAME
                  --font-properties FONT_PROPERTIES
                  [--experience_number EXPERIENCE_NUMBER]
                  [--font-size FONT_SIZE]
                  [--tessdata-path TESSDATA_PATH]
                  [--word_list WORD_LIST]
                  [--verbose]
```

**tesstrain arguments**

	  -h, --help            show this help message and exit

	**Required arguments:**
	  --tesseract-lang TESSERACT_LANG, -l TESSERACT_LANG
	                        Set the tesseract language traineddata to create.
	  --training-text TRAINING_TEXT, -t TRAINING_TEXT
	                        The path of the training text.
	  --font-path FONT_PATH, -F FONT_PATH
	                        The path of TrueType/OpenType file of the used training font.
	  --font-name FONT_NAME, -n FONT_NAME
	                        The name of the used training font. No spaces.
      --font-properties FONT_PROPERTIES, -f FONT_PROPERTIES
                            The path of a file containing font properties for a list of training fonts.

	**Optional arguments**
	  --experience_number EXPERIENCE_NUMBER, -e EXPERIENCE_NUMBER
	                        The number of the training experience.
	                        Default value: 0
	  --font-size FONT_SIZE, -s FONT_SIZE
	                        The font size of the training font, in px.
	                        Default value: 25
	  --tessdata-path TESSDATA_PATH, -p TESSDATA_PATH
	                        The path of the tessdata/ directory on your filesystem.
	                        Default value: /usr/local/share/tessdata
	  --word_list WORD_LIST, -w WORD_LIST
	                        The path of a file containing a list of frequent words.
	                        Default value: None
	  --verbose, -v         Use this argument if you want to display the training
                            output.


### Examples

In this example, we would like to create a `helveticanarrow` dictionary:

* using an OpenType file located at `./font/Helvetica-Narrow.otf`
* the font name is set to `helveticanarrow`
* with training text located at `./text`
* the `font_properties` file is located at `./font_properties`. It contains the following line: `helveticanarrow 0 0 0 0 0`
* the experience number is set to 0
* a tif font size of 25px
* the `tessdata` directory is located at `/usr/local/share/tessdata`
* no frequent word list

The command would thus be:

```bash
$ tesstrain --tesseract-lang helveticanarrow --training-text ./text --font-path font/Helvetica-Narrow.otf --font-name helveticanarrow  --font-properties ./font_properties --verbose
```
or using the short options names:

```bash
$ tesstrain -l helveticanarrow -t ./text -F ./font/Helvetica-Narrow.otf -n helveticanarrow -f ./font_properties -v
```

## Python API

The `tesseract_train.py` file offers a very simple API, defined through the class `TesseractTrainer`.
This class has only 4 public methods:

* `__init__(self, text, exp_number, dictionary_name, font_name, font_size, font_path, font_properties, tessdata_path, word_list)`: returns a `TesseractTrainer` instance
* `training(self)`: performs all training operations, thus creating a `traineddata` file.
* `add_trained_data(self)`: copies the generated `traineddata` file to your `tessdata` directory
* `clean(self)`: deletes all files generated during the training process (except for the `traineddata` one).

### Example

```python
from tesseract_trainer import TesseractTrainer

trainer = TesseractTrainer(dictionary_name='helveticanarrow',
                            text='./text',
                            font_name='helveticanarrow',
                            font_properties='./font_properties',
                            font_path='./font/Helvetica-Narrow.otf')
trainer.training()  # generate a multipage tif from args.training_text, train on it and generate a traineddata file
trainer.clean()  # remove all files generated in the training process (except the traineddata file)
trainer.add_trained_data()  # copy the traineddata file to the tessdata/ directory
```

Note that the same default values apply than when using the `tesstrain` command:

```python
font_size = 25
exp_number = 0
tessdata_path = "/usr/local/share/tessdata"
word_list = None
verbose = True
```

You can override these constants when instanciating a `TesseractTrainer` object, to better suit your needs.

## Remarks

* UTF-8 encoding is supported.
* If your `tessdata` directory is not writable without superuser rights, use the `sudo` command when executing your python script.
* Do not forget to describe your font properties in a file, following [these instructions](https://code.google.com/p/tesseract-ocr/wiki/TrainingTesseract3#font_properties_%28new_in_3.01%29).

## Installation troubleshoot
### PIL installed without JPEG, PNG and freetype support in a virtualenv
If you install TesseractTrainer in a virtualenv, PIL will have to be installed along.
A well known problem is that it might not be installed with JPEG, PNG and freetype support (see this [thread](http://ubuntuforums.org/showthread.php?t=1751455)).

In that case, follow these [instructions](http://ubuntuforums.org/showthread.php?t=1751455), to have a working PIL installation in
your virtualenv.


## Attributions
TesseractTrainer was completed whilst working on [StrongSteam](http://strongsteam.com) for [MorConsulting](http://morconsulting.com/).
