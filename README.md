<p align="center">
<img width=30% src="https://dai.lids.mit.edu/wp-content/uploads/2018/08/orion.png" alt=“Orion” />
</p>

<p align="center">
<i>Orion is a machine learning library built for data generated by satellites.</I>
</p>

<p align="center">
<i>An open source project from Data to AI Lab at MIT.</I>
</p>

# Orion
Orion is a machine learning library built for sensor data collected from Satellites. The library makes use of a number of automated machine learning tools developed under ["The human data interaction project"](https://github.com/HDI-Project)  within the [Data to AI Lab at MIT](https://dai.lids.mit.edu/). The focus, with the ready availability of automated machine leanring is on:

* domain expert interaction with the machine learning system
* learning from minimal labels
* explainability of model outputs
* model audit
* scalability

## License
- MIT license


# Install

To install the project, after cloning the repository and creating a virtualenv, execute

```
make install
```

For development, use the following command instead, which will install some additional
dependencies for code linting and testing

```
make install-develop
```

# Command Line Interface usage example

In the following example we load the `S-1` signal from NASA data,
execute the `lstm_dynamic_threshold` pipeline on it and store the
found anomalies in the database.

Afterwards, we use the CLI to insert and see some comments.

1. First, we reset the database to start from scratch.
   This command will ask the user to input the name of the database
   as a safe mesure.

```
$ orion reset
WARNING: This will drop the database!
Please enter the database name to confirm: orion
Dropping database orion
```

2. Afterwards we have no datasets and no pipelines registered, so we proceed to register
   one of the NASA signals and a couple of pipelines

```
$ orion list datasets
No datasets found
$ orion add dataset S-1 --user 1
$ orion list datasets
                 dataset_id created_by                data_location             insert_time           name     signal_set  start_time   stop_time  timestamp_column  value_column
0  5c7d43f96c1cea61a996b205          1                          NaN 2019-03-04 15:27:53.397            S-1            S-1  1222819200  1442016000                 0             0
$ orion list pipelines
No pipelines found
$ orion add pipeline nasa orion/pipelines/lstm_dynamic_threshold.json -u 1
$ orion list pipelines
                pipeline_id created_by             insert_time  name
0  5c7d44006c1cea61d0f7d9bb          1 2019-03-04 15:28:00.436  nasa
```

3. Once we have a dataset and a pipeline registered, we can run an analysis.

```
$ orion run nasa S-1 -v -u 1
2019-03-04 16:28:13,073 - 25115 - INFO - explorer - Loading dataset S-1
2019-03-04 16:28:13,084 - 25115 - INFO - explorer - Loading pipeline nasa
Using TensorFlow backend.
2019-03-04 16:28:14,015 - 25115 - INFO - analysis - Fitting the pipeline
Epoch 1/1
2019-03-04 16:28:18.235383: I tensorflow/core/platform/cpu_feature_guard.cc:141] Your CPU supports instructions that this TensorFlow binary was not compiled to use: AVX2 FMA
2019-03-04 16:28:18.256606: I tensorflow/core/platform/profile_utils/cpu_utils.cc:94] CPU Frequency: 2208000000 Hz
2019-03-04 16:28:18.257409: I tensorflow/compiler/xla/service/service.cc:150] XLA service 0x1482210 executing computations on platform Host. Devices:
2019-03-04 16:28:18.257428: I tensorflow/compiler/xla/service/service.cc:158]   StreamExecutor device (0): <undefined>, <undefined>
9899/9899 [==============================] - 55s 6ms/step - loss: 0.0561 - mean_squared_error: 0.0561
2019-03-04 16:29:23,237 - 25115 - INFO - analysis - Finding events
/home/xals/.virtualenvs/Orion/lib/python3.6/site-packages/scipy/optimize/optimize.py:570: RuntimeWarning: invalid value encountered in subtract
  numpy.max(numpy.abs(fsim[0] - fsim[1:])) <= fatol):
2019-03-04 16:29:37,693 - 25115 - INFO - analysis - 1 events found in 0:01:23.728467
Datarun id: 5c7d440d6c1cea621b80e879
```

4. Once the pipeline has finished, we can see a summary of our datarun.

```
$ orion list dataruns
                 datarun_id created_by                   dataset                end_time  events             insert_time                  pipeline              start_time status
0  5c7d440d6c1cea621b80e879          1  5c7d43f96c1cea61a996b205 2019-03-04 15:29:37.688       1 2019-03-04 15:28:13.961  5c7d44006c1cea61d0f7d9bb 2019-03-04 15:28:13.960   done
```

5. And we can see the events that our datarun detected.

```
$ orion list events -d 5c7d440d6c1cea621b80e879
                   event_id                   datarun             insert_time     score  start_time   stop_time  comments
0  5c7d44616c1cea621b80e87a  5c7d440d6c1cea621b80e879 2019-03-04 15:29:37.636  0.017108  1399075200  1399183200         0
```

6. Using the `event_ids`, we can add some comments to individual events.

```
$ orion add comment 5c7d44616c1cea621b80e87a -u 1 "This is a comment"
$ orion list events -d 5c7d440d6c1cea621b80e879
                   event_id                   datarun             insert_time     score  start_time   stop_time  comments
0  5c7d44616c1cea621b80e87a  5c7d440d6c1cea621b80e879 2019-03-04 15:29:37.636  0.017108  1399075200  1399183200         1
$ orion list comments
                   event_id                   datarun     score  start_time   stop_time                comment_id created_by             insert_time               text
0  5c7d44616c1cea621b80e87a  5c7d440d6c1cea621b80e879  0.017108  1399075200  1399183200  5c7d451b6c1cea670c1eeeec          1 2019-03-04 15:32:43.427  This is a comment
```

7. We can even add more than one comment for each event

```
$ orion list comments
                   event_id                   datarun     score  start_time   stop_time                comment_id created_by             insert_time                       text
0  5c7d44616c1cea621b80e87a  5c7d440d6c1cea621b80e879  0.017108  1399075200  1399183200  5c7d451b6c1cea670c1eeeec          1 2019-03-04 15:32:43.427          This is a comment
1  5c7d44616c1cea621b80e87a  5c7d440d6c1cea621b80e879  0.017108  1399075200  1399183200  5c7d45456c1cea67e75ff403          1 2019-03-04 15:33:25.088  This is a another comment
```

8. Optionally, each of these outputs can be exported as a CSV file:

```
$ orion list events -d 5c7d440d6c1cea621b80e879 -o events.csv
Storing results in events.csv
$ cat events.csv 
event_id,datarun,insert_time,score,start_time,stop_time,comments
5c7d44616c1cea621b80e87a,5c7d440d6c1cea621b80e879,2019-03-04 15:29:37.636,0.017108323858931792,1399075200,1399183200,2
```
