# P State Visualizer

A Python 3 script to parse the log file from a P test, and display a grid, colored with ANSI escape sequences.

## Running
To parse a log file, make sure your terminal window is very wide, and run the script with the log filename
as the argument:
```
python3 state_visualizer.py POutput/netcoreapp3.1/Output/TwoPhaseCommit.dll/CoyoteOutput/TwoPhaseCommit_0_12.txt
```

By default, the column width is 20. If you want to specify an alternate column width, use the --column-width argument:
```
python3 state_visualizer.py --column-width=25 POutput/netcoreapp3.1/Output/TwoPhaseCommit.dll/CoyoteOutput/TwoPhaseCommit_0_12.txt
```
or
```
python3 state_visualizer.py --column-width 25 POutput/netcoreapp3.1/Output/TwoPhaseCommit.dll/CoyoteOutput/TwoPhaseCommit_0_12.txt
```


