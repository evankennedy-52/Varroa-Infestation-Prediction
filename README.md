# Varroa-Infestation-Prediction
**Author:** Evan Kennedy


## Repository Description
This repository houses utilities to create a predictive model for Varroa mite infestations of *Apis mellifera* (Western honeybee) colonies.

The intended use of this project is to enable users to input data on their own apiaries, and on apiaries similar to their own (i.e., geographically close), to receive a tailor-made model specifically intended to predict Varroa mite infestations within apiaries like theirs.

Included in the repository is data from hives in Austria, provided as an example. Models fit on this data are not intended to be used for apiaries outside of Austria, but can be if a user lacks data, though performance may be less reliable.
 - Example Data Source: https://data-for-good.pubpub.org/pub/aawkkv33/release/4

For reference, on the example data, the model produced by this project's pipeline had a prediction accuracy of 95% for hives in Austria.
 - This is the model yielded if the pipeline is run (explained later) without changing any configurations (explained later)


## Getting Your Own Copy
The simplest way to use this tool is to download the repository as a ZIP file and host it locally on your own machine.

To download this repository as a ZIP file, click the green 'Code' button at the top right of the repository's main page and select 'Download ZIP'.

More advanced users may also clone the repository via git for local use, but should be aware that pushes to the repository are not allowed without admin permission.
 - Note: Cloning the repository is NOT necessary to ensure the latest functionalities are being used locally


## Using this Repository (Quick Start: Recommended to Read Below First)
To use the main utilities of the project, pipeline.yml and predict.yml, two prerequisite steps are required:
 - Docker Desktop must be open in the background. This is necessary to enable the Docker daemon that the Docker Compose files use.
     - If help is needed: https://www.docker.com/products/docker-desktop/
 - ClearML API credentials must be configured for the system. These are used for logging model performance in the ClearML Web UI.
     - If help is needed: https://clear.ml/docs/latest/docs/webapp/settings/webapp_settings_profile/

Once these prerequisites are satisfied, pipeline.yml and predict.yml can be run using the methods described above, for the following results
(all based on the configurations specified in the /configs files, as explained above):
 - pipeline.yml:
    - Tensors data file in pipeline/outputs [Not for user interaction]
    - Model file in pipeline/outputs [Not for user interaction]
    - A prediction in the 'predictions' folder (the folder will appear if it was absent)
    - Model performance metrics in the ClearML Web UI, in the project 'Varroa Infestation Prediction', within the task 'XGBoost Optuna HPO'
 - predict.yml (if run properly, only after pipeline.yml has already created a model):
    - A prediction in the 'predictions' folder (the folder will appear if it was absent)


## Repository Components
### /data
A folder containing 6 .csv files. These files are included as a demonstration, and are the files used as reference when the model was originally developed.

Data from a user's own operation should be placed in this folder and specified in the appropriate config files (explained below) to be used for model creation and/or making predictions. Appropriate formatting is as follows:
 - **Weather Data**: Time-series CSV with fields ["station_id","date","hour","air_temp","dew_point","pressure","wind_dir","wind_spd"]
    - *station_id*: Weather station identifier. Can be from a weather service or proprietary labeling, but must be an integer value
    - *date*: XXXX-XX-XX string, encoding Year-Month-Day
    - *hour*: XX:XX:XX string, encoding Hour:Minute:Second
    - *All Other Fields (i.e., Weather Measuring Fields)*: Floating point numbers (i.e., decimals allowed), using -9999.0 to represent a missing value
    - For reference, see: weather.csv
 - **Varroa Sampling Data**: CSV with ["sampling_id","date_from","date_to","varroa_count","yard_id"]
    - *sampling_id*: Unique identifier for a sampling count of Varroa mites. Must be an integer value
    - *date_from*: Date when the drop board was freshly set, prior to the sampling: XXXX-XX-XX string, encoding Year-Month-Day
    - *date_to*: Date when the mites were counted: XXXX-XX-XX string, encoding Year-Month-Day
    - *varroa_count*: Manual count of Varroa mites on the board. Must be an integer value
    - *yard_id*: A unique integer identifier for a particular yard/apiary
    - For reference, see: varroa_sampling.csv
 - **Yard Data**: CSV with ["yard_id","elevation","station_id"]
    - *yard_id*: A unique integer identifier for a particular yard/apiary. Should match those used in the selected Varroa Sampling Data
    - *elevation*: Floating point value (i.e., decimals allowed) denoting the elevation of the particular yard/apiary
    - *station_id*: Closest weather station identifier. Should match those used in the selected Weather Data
    - For reference, see: yard.csv

IMPORTANT: User-included data files should be given names without spaces (i.e., replace spaces with an underscore, `_`).

Note: The easiest way to turn records into CSV files is often to use Excel's 'Save As' (or 'Export') option to convert an Excel workbook into a CSV file.

### /configs
A folder with 2 JSON files in it, used for inputs to phases of the pipeline:
 - **pipeline_config.json**: Inputs to the model-creation segment of the pipeline
     - *model_name*: What you would like the model produced to be called, useful primarily if creating multiple models from different data
         - FORMATTING REQUIREMENT: In quotes (i.e., "name"), ends in .json file extension
     - *varroa_samplings_file_path*: Path to the desired Varroa sampling data, relative to the base directory of the project
         - FORMATTING REQUIREMENT: In quotes (i.e., "name")
     - *yard_information_file_path*: Path to the desired yard information data, relative to the base directory of the project
         - FORMATTING REQUIREMENT: In quotes (i.e., "name")
     - *weather_data_file_path*: Path to the desired weather data, relative to the base directory of the project
         - FORMATTING REQUIREMENT: In quotes (i.e., "name")
 - **prediction_config.json**: Inputs to the prediction segment of the pipeline
     - *predictor_model_name*: What model you would like to use to make a prediction. Matches the created model name by default
         - FORMATTING REQUIREMENT: In quotes (i.e., "name"), ends in .json file extension
     - *weather_data_file_path*: Path to the desired weather data (historical or forecast), relative to the base directory of the project
         - FORMATTING REQUIREMENT: In quotes (i.e., "name")
     - *weather_station_id*: The identifier for the weather station closest to the yard/apiary having a prediction made for it
         - FORMATTING REQUIREMENT: In quotes (i.e., "name")
     - *yard_density*: An integer value for the number of hives in the yard/apiary having a prediction made for it
         - FORMATTING REQUIREMENT: Integer value
     - *yard_elevation*: A floating point value (i.e., decimals allowed) for the elevation of the yard/apiary having a prediction made for it
         - FORMATTING REQUIREMENT: Floating point value
     - *prediction_period_start_date*: XXXX-XX-XX string, encoding Year-Month-Day, for the start of the date range to make predictions for
         - FORMATTING REQUIREMENT: In quotes (i.e., "XXXX-XX-XX")
     - *prediction_period_end_date*: XXXX-XX-XX string, encoding Year-Month-Day, for the end of the date range to make predictions for
         - FORMATTING REQUIREMENT: In quotes (i.e., "XXXX-XX-XX")

These files can be opened with a standard text editor to change input parameters. Be mindful that the parameters are the values to the right of the ':' on each line, before the comma. When adjusting parameters, be sure to keep the comma and ':' in place, and to match the format of the values already in place by default (specified above).

### /pipeline
Contains the three Jupyter notebooks that form the three phases of the pipeline:
 - **Data Processor**: Processes the specified data files into combined tensors
 - **Model Fitter**: Fits a predictive model to the data
 - **Predictor**: Makes a true/false prediction of infestation, accompanied by a probability of infestation, for a given date range

Additionally includes a folder labeled 'outputs' with an empty 'dummy.txt' file in it. The folder is important to have in place; the file inside can be removed if desired.

Generally speaking, users should not interact directly with any of this folder's contents.

Note: While these notebooks can be run in sequence to simulate the pipeline, it is not the intended approach, and can fail or produce unintended behavior if the local environment does not match the requirements laid out in Requirements.txt and is not running Python 3.11.9.

### pipeline.yml
A Docker Compose file used to run the pipeline through all of its phases, producing a predictive model and predictions based on the configurations specified in the /configs files.

Running options:
 - *On Windows*: Double-click the **pipeline.bat** file
 - *Non-Windows Systems*: Navigate to the project directory in a Terminal/Command-Line Interface, then run:
   ```
   docker compose -f pipeline.yml build --no-cache && docker compose -f pipeline.yml up --remove-orphans
   ```
      - Note: The easiest way to navigate to the project directory is with `cd {full project directory path}`, pasting in the directory path as appropriate

### predict.yml
A Docker Compose file used to run the predictor. It ONLY works after pipeline.yml has already been run at least once to produce the model the predictor is specified to use (by the /configs files).

Running options:
 - *On Windows*: Double-click the **predict.bat** file
 - *Non-Windows Systems*: Navigate to the project directory in a Terminal/Command-Line Interface, then run:
   ```
   docker compose -f predict.yml build --no-cache && docker compose -f predict.yml up --remove-orphans
   ```
      - Note: The easiest way to navigate to the project directory is with `cd {full project directory path}`, pasting in the directory path as appropriate

### pipeline.bat & predict.bat
Windows shortcuts to run the Docker Compose files the project is built on (use explained above).

### /utilities
A folder with a supporting file included in it. It should not be interacted with by the user.

### .gitattributes & .gitignore
Files included for the GitHub integration of the project, not to be interacted with by the user.

### Dockerfile
A file used in running the pipeline and the predictor, not interacted with directly by the user.

### Requirements.txt
A text file encoding the installation requirements for running the pipeline. It is not necessary for the user to interact with it.

Note: Can be used as a reference if wishing to run the notebooks individually within a virtual environment, though this approach is not the recommended one.
