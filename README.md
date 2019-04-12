# Keyboard Acoustic Emanations Attack (Research)

## Objectives
  * Create a proof-of-concept for a keyboard acoustic emanations attack, and evaluate
    its __accuracy__, __effectiveness__, and __accessibility__.
  * Keyboard acoustic emanations attack: Given just the sound of a victim typing on a keyboard,
    an attacker analyzes and extracts what was typed.
  * Theoretically requires no supervised training, and instead uses unsupervised methods to
    approximate typed keys according to various methods (view paper below).

## Motivation
Many research papers were published in the mid-2000s concerning the topic of keyboard acoustic
emanations attacks. Some research, such as [*Keyboard Acoustic Emanations Revisited* by L. Zhuang,
F. Zhou, J. D. Tygar in 2005](https://www.cs.cornell.edu/~shmat/courses/cs6431/zhuang.pdf), demonstrated
extremely accurate results (96% chars recovered from 10 minute sound recording) even without labeled 
training data. Technology surrounding machine learning has advanced considerably since the time such
research was published. This project therefore aims to identify how __accurate__, __effective__, and
__accessible__ such a security attack is in the current machine learning landscape. (If an undergraduate
student researcher such as myself can create a relatively effective prototype, this would likely be a
sign for a considerable security concern.)

## Setting up
### Option 1 - Docker (recommended)
This project uses a Python 3.6 development environment and a PostgreSQL database
to manage various audio data. These environments are spun up using Docker Compose.  
* Install Docker. (https://www.docker.com/products/docker-desktop)  
* Build image (only required the first time or whenever Docker settings are changed) with

        docker-compose build
        
This step will install all dependencies for env (such as Jupyter, Tensorflow, NumPy etc.)
and mount your local file system with the file system within the "env" Docker container.
        
* Spin up the database and development environment with
  
        docker-compose up
        
This should open up the database for connections and make *localhost* (port 8888) access
the Jupyter notebook.

### Option 2 - No Docker
This option isn't really recommended, but I'll leave it here just in case Docker isn't an option
for whatever reason.

Install Python version 3.6.  
Set up virtual environment. I recommend virtualenvwrapper for managing environments.   
Install dependencies with `pip3 install -r requirements.txt`.  
Open Jupyter notebook with `jupyter notebook`.


This option is simpler if you're unfamiliar with Docker or you don't need to access the database.
(Though the latter should still be possible using local postgres commands)


## Development
__Disclaimer__: Because this project is still in its very early stages and the repository is prone to frequent and
drastic refactorings, this section is likely to change the most. Many of the instructions or file locations
listed here may be completely wrong (though I'll do my best to actively keep this up-to-date).

### File I/O, keystroke extraction
The current method of processing sound data is recording audio, saving in WAV format, and extracting the keystroke
sounds as NumPy arrays. Functions to handle these processes are located in __src/scripts/audio_processing.py__.

### Database storage and retrieval
To avoid having the parse WAV files for keystrokes each time data is required, extracted keystroke data is stored
in a Postgres database (via SQLAlchemy). When using the data for training a model, the data is first retrieved from the 
database then preprocessed externally. Functions for storing and loading data is also located in
__src/scripts/audio_processing.py__.

### Other
Example for a basic keystroke classifier (currently at an incredible 4-5% accuracy at the time of writing) is located
__src/scripts/supervised_learning.py__. There are also accompanying Jupyter notebooks for both __audio_processing.py__
and __supervised_learning.py__ in the __src/jupyter__ directory. These notebooks (for the most part) mirror the code in
the corresponding __src/scripts__ directory.

In many cases, I find it useful to run a Python script within the Docker container specified by __Dockerfile__, but outside of
the Jupyter notebook (I've had cases where the Jupyter kernel dies during computationally intensive work, but work fine when
the same operations are run outside of the notebook). In such cases, run

    docker-compose run env <insert Python command>
    
Example:
    
    docker-compose run env python -i src/scripts/supervised_learning.py

## Relevant Research Papers
Research papers for reference. For the most part, I intend to follow the methodology for the Zhuang, Zhou, Tygar paper 
(k-clustering, HMMs for guessing, iterating with trained classifier from inferred data).

### Supervised Methods
  * [*Keyboard acoustic emanations*](https://ieeexplore.ieee.org/document/1301311)
    by D. Asonov, R. Agrawal. 2004.

### Unsupervised Methods
  * [*Keyboard Acoustic Emanations Revisited*](https://www.cs.cornell.edu/~shmat/courses/cs6431/zhuang.pdf)
  by L. Zhuang, F. Zhou, J. D. Tygar. 2005.
  * [*Dictionary Attacks Using Keyboard Acoustic Emanations*](https://www.eng.tau.ac.il/~yash/p245-berger.pdf)
  by Y. Berger, A. Wool, A. Yeredor. 2006.

## Notes (to self)
  * The Dockerfile in this repository runs `jupyter notebook` with the `--allow-root` option
    to bypass errors the lazy way. This isn't really advised, so this should probably be
    reconsidered if this Docker container is ever run on a server.
