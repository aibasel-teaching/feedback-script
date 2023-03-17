# ADAM Correction Script
The purpose of this script is to automate some of the menial steps involved in
marking ADAM submissions.

The system is made up of two components, a JSON configuration file called
`config.json`, and the central Python script `adam-script.py`.

The config file has a general parameters that need to be adapted to the course
that's being taught, and individual parameters that need to be set by every
tutor. This file has to be edited once, at the beginning of the course, and
should thereafter remain the same for the rest of the semester.
The script then takes the settings from the config file, as well as additional
command line parameters, to simplify some steps in the marking process.

Depending on the parameters set by the config file, the the necessary command
line options at every step may vary. In the following I will showcase the
workflow using the settings of Foundations of Artificial Intelligence SS23 as an
example.

## Requirements
- `Python 3.10+`: I only tested the script with 3.10 and I think it makes use of
some new-ish language features, so I'm not sure if everything works with older
versions.

## One-Time Setup
To get started, create a directory where you want to do your marking, in this
example the directory will be called `ki-fs23-marking`. In this directory you
place the `config.json` file you should have gotten from the teaching assistant,
and fill in the individual settings with your own information, the relevant
fields should be indicated in the file. Alternatively you can find a complete
list [here](##-Individual-Settings). Make sure that the string you enter in the
field `your_name` exactly matches the existing entry in `tutor_list`.

In general, it is important that the all configurations, besides the individual
ones you just adjusted, are exactly the same across all tutors, as otherwise
submissions may be assigned to multiple or no tutors. If you think that
something should be changed in the shared settings, please let the teaching
assistant and the other tutors know, so that the configurations remain in sync.
This may be in particular be necessary if teams change during the semester.

In order to work with the script, you will have to call `adam-script.py` from a
command line whose working directory is the one which contains the `config.json`
file. This means that you should keep `adam-script.py` somewhere where it's easy
to access. On my Linux machine, I would do this by leaving the script in the
repository clone and adding a symbolic link pointing to its location to the
`ki-fs23-marking` directory via the following command:
```
ln -s /path/to/adam-script.py .
```
This way I can pull the newest version of the script and have it available in
the right place without any extra steps. Alternatively you could do your marking
directly in the directory of the script repo clone or paste a copy of the script
to the marking directory.

You should now be able to run the script from the command line, for example to
print the help text. On my setup I would run:
```
./adam-script -h
```
If you are, for example, using windows and have the script in the parent
directory, you'd do something like
```
python3 ..\adam-script.py -h
```

## Marking a Sheet
While the steps above are only necessary for the initial setup, the following
procedure applies to every exercise sheet.

### init
First, download the submissions from ADAM and save the zip file in the marking
directory. (It's important that you only download the submissions after the
ADAM deadline has passed, so that all tutors have the same pool of submissions.)
Our example directory `ki-fs23-marking`, with `Sheet 1.zip` being the file
downloaded from ADAM, should look like this:
```
.
├── adam-script.py
├── config.json
└── Sheet 1.zip
```
We can now finally make the script do something useful by running:
```
./adam-script.py init -n 4 -t sheet01 "Sheet 1.zip"
```
This will unzip the submissions and prepare them for marking. The flag `-n`
expects the number of exercises in the sheet, `-t` is optional and takes the
name of the directory the submissions should be extracted to. By default it's
the name of the zip file. The directory should now look something like this:
```
.
├── adam-script.py
├── config.json
├── sheet01
│   ├── 12345_Muster_Müller
│   │   ├── feedback
│   │   │   └── feedback_tutor-name.pdf.todo
│   │   └── Sheet1_MaxMuster_MayaMueller.pdf
│   .
│   ├── DO_NOT_MARK_12346_Meier_Meyer
│   │   └── submission_exercise_sheet1.pdf
│   .
│   └── points.json
└── Sheet 1.zip
```
As you may have guessed, the submissions you need to mark are those without the
`DO_NOT_MARK_` prefix. Those directories contain the files submitted by the
respective team, as well as a directory called `feedback`, which in turn
contains an empty placeholder PDF file and copies of submitted files that are
not PDFs (e.g. source files).

The idea is that you can give feedback to non-PDFs by adding your comments to
these copies directly, and delete the ones you don't need to comment on. For the
PDF feedback you can use whichever tool you like, and overwrite the `.pdf.todo`
placeholder with the resulting output. If this tool adds files to the feedback
directory that you do not want to send to the students, you can add their
endings to the config file under the `ignore_feedback_suffix` key. Marking with
Xournal++ is supported by default, add the flag `-x` to the `init` command above
to automatically create the relevant `.xopp` files.

While writing the feedback, you can keep track of the points the teams get in
the file `points.json`.

### collect
Once you have marked all the teams assigned to you and added their points to
the `.json` file, you can run the next command, where `sheet01` is the path to
the directory created by the `init` command:
```
./adam-script.py collect sheet01
```
This will create a zip archive in every feedback directory containing the
feedback for that team. Additionally, a semicolon-separated list of all points
is printed. This can be pasted to the spreadsheet where all points are
collected. The names are only there to be able to double-check that the rows
match up.

In case you need make changes to the markings and rerun the collection step, use
the `-r` flag to overwrite existing feedback archives. If you are using
Xournal++, you can also use the `-x` flag here to automatically export the
`.xopp` files before collecting the feedback.

### send
In the future, the script should be able to directly send out feedback per mail,
but this is not possible yet `:(`. For now, you still have to manually upload
the feedback zip archives to ADAM.



## Setup Config File

### Individual Settings
- `your_name`: ID of the tutor, this must match with either an element of
  `tutor_list` (for `random/exercise`) or a key in `teams` (for `static`)
- `ignore_feedback_suffix`: a list of extensions that should be ignored by the
  `collect` sub-command; this is useful if the tools you use for marking create
  files in the feedback folders that you don't want to send to the students

### Email Settings
- `your_email`: tutor's email address, feedback will be sent via this address
- `feedback_email_cc`: list of email addresses that will be CC'd with every
  feedback email, for example the addresses of all tutors
- `smtp_url`: the url of the smtp server, `smtp-ext.unibas.ch` by default
- `smtp_port`: smtp port to connect to, `587` by default
- `smtp_user`: smtp user, usually the short Unibas account name

### General Settings
- `lecture_title`: lecture name to be printed in feedback emails
- `marking_mode`
    - `static`: student teams are assigned to a tutor who will mark all their
      submissions
    - `random`: student teams are randomly assigned to tutors with every sheet,
      this tutor will mark the team's entire submission
    - `exercise`: with every sheet, tutors distribute the exercises and only
      correct those, but for all submissions
- `points_per`
    - `exercise`: tutors keep track how many points teams got for every exercise
    - `sheet`: tutors only keep track of the total number of points per sheet
- `min_point_unit`: a float denoting the smallest allowed point fraction, for
  example `0.5`, or `1`
- `tutor_list`: list to identify tutors, for example a list of first names
- `max_team_size`: integer denoting the maximum number of members a team may
  have
- `teams`: depending on the `marking_mode` teams are structured slightly
  differently
    - `random/exercise`: list of teams, each consisting of a list of students,
      where each student entry is a list of the form `[ "first_name",
      "last_name", "email@unibas.ch" ]`
    - `static`: similar to before, but teams are not just listed, but assigned
      to a tutor; this is done via a dictionary where some ID for the tutors
      (e.g. first names) are the keys, and the values are the list of teams
      assigned to each tutor
