# Spectre tips for beginners
This repo holds tips and notes for how to do useful things when getting up to speed with [spectre](spectre-code.org).

# install code
1. Clone the Spectre repository from GitHub:
`git clone git@github.com:sxs-collaboration/spectre.git`

2. Retrieve the docker image:
`docker pull sxscollaboration/spectrebuildenv:latest`

3. Start the docker container
`docker run -v /Your/path/to/Spectre/:/SPECTRE_ROOT --name CONTAINER_NAME -i -t sxscollaboration/spectrebuildenv:latest /bin/bash`

Just need to replace the above command with the full path to your clone of Spectre
(The --name CONTAINER_NAME is optional, where CONTAINER_NAME is a name of your choice. If you don't name your container, docker will generate an arbitrary name.) You will end up in a shell in a docker container, as root (you need to be root). For the following steps, stay inside the docker container as root.

4. Make a build directory somewhere inside the container, (e.g. /work/spectre-build-gcc), and cd into it. (You should already be in /work/)
`mkdir spectre-build-gcc`

5. Build Spectre:
`cmake -D CHARM_ROOT=/work/charm/multicore-linux64-gcc /SPECTRE_ROOT/` where /SPECTRE_ROOT/ is the path in step 3

6.  `make -jN`
(Replace N with the number of cores to use)
If on your macbook, try `make -j1`

7.`make test-executables -jN` (to compile the test executables)

8. `ctest` (to run the tests.)

# Tips for writing new spectre code
## Things to do before making a pull request to merge into `sxs-collaboration/spectre`

  - Format your code with `git-clang-format-5.0`
  - Make sure your code compiles by running `make -j2` in your build directory 
  (in docker, typically `/work/spectre-build-clang`). Replace `2` with the number of processors if you're on a machine with 
  more than 2 cores, like a high-end Mac/PC or a cluster. If the process hangs, you might have run out of memory; in that 
  case, try using fewer cores. Fix any compiler errors and warnings you get. 
  - Also in your build directory, make sure all tests pass with `ctest -j2` where 2 is the number of processors.
  - For each `*.cpp` file you created or modified, in your build directory, run `make clang-tidy FILE=/path/to/file.cpp`. 
  [Clang-tidy](http://clang.llvm.org/extra/clang-tidy/) is a "linter", a tool that helps to find and fix common 
  programming mistakes that the compiler won't catch with a compiler error.
  - Make a second build directory, e.g. `/work/iwyu-spectre-build-clang` and `cd` into it. Configure as usual, but add the 
  option `-DUSE_PCH=OFF` to `cmake`. Then, in the second build directory, for each `*.cpp` file, 
  run `make iwyu FILE=/path/to/*.cpp`. This will check that your `#include` lines. If you get errors from this, check 
  with an experienced spectre developer, as these error messgaes sometimes are wrong.
  - In your normal build directory, run `make doc`. Copy thre `docs/html` directory outside of docker (if you are using 
  docker), and then open `docs/html/index.html` in your browser. Make sure documentation you made looks right
  

## How to format your code
For each file you have changed (`git status` before commit, or look in github after commit), do this, replacing `/path/to/file` with the path to the file you want to format:
~~~~
git-clang-format-5.0 -f /path/to/file
~~~~
This will automagically format only the lines you changed to google style. If you want to reformat an entire file 
(useful if you made a brand new file), just do 
~~~~
clang-format -i /path/to/file
~~~~

# Git related code snippets etc.

## How to update your fork when the code you forked from changes
Here is how you update the develop branch of your fork to get the latest changes from the main spectre repo (called the "upstream repo."
~~~~
# Set up upstream (first time only)
git remote add upstream ORIGINAL-REPO-LOCATOR
# Update a fork
git fetch upstream
git checkout develop
git merge upstream/develop
git push
~~~~

# Docker related things
To see all your containers: `docker ps -a`

To remove a container: `docker rm --force NameOfContainer`

To start docker: `docker start -i ContainerName` or `docker start -i ContainerID`

# Visualizing Waves on Paraview (with OCEAN)
1. On ocean, make sure you have compilied spectre.

2. In spectre, `cd build`, then `make EvolveScalarWave3D`. EvolveScalarWave3D is just a program like cd, ls, ln -s, etc.
  - If this isn't working then maybe you didn't load your OCEAN environment and spectre modules. <br />
      Try `. $SPECTRE_HOME/support/Environments/ocean_gcc.sh` and `spectre_load_modules`. <br />
  - Still not working? Maybe you didn't compile spectre.<br />
      Try `make -j10`.<br />
      
3. Somewhere NOT in your spectre directory, make a new directory for your wave tests and cd into it. For this example, I will call this directory WaveTest. 

4. Here we're going to set up an enviroment to run the input file and obtain output files. <br />
      `cp /YourPathToSpectre/tests/InputFiles/ScalarWave/Input3DPeriodic.yaml .`<br />
      `ln -s /YourPathToSpectre/build/bin/EvolveScalarWave3D .`
      
5. Now we can edit out input file, Input3DPeriodic.yaml.
    *edits input file*
    
6. Once we have edited Input3DPeriodic.yaml, we excute it,<br />
      `./EvolveScalarWave3D --input-file Input3DPeriodic.yaml` <br />
  - You should now see two output files (ScalarWave3DPeriodicReductions.h5 and ScalarWave3DPeriodicVolume0.h5). If you didn't,      then checkout https://spectre-code.org/tutorial_visualization.html (specifically the Running an Evolution Executable section).

4. We have H5 files, however ParaView cannot read these so we need to convert the ScalarWave3DPeriodicVolume0.h5 into a .xmf<br />
  - In order to do this we need to copy the scripts: `ln -s /YourPathToSpectre/tools/GenerateXdmf.py .` <br />
  - Next we need to run the script: `python GenerateXdmf.py --file-prefix ScalarWave3DPeriodicVolume --output ScalarWave3DPeriodicVolume` and if this doesn't work, look at `python GenerateXdmf.py -h` for the help text.

5. Everything is complete, now we just need to copy all the ScalarWave3DPeriodicVolume files onto our local machine so we can use Paraview. In the command below, "PathToWaveTestDirectory" refers to step 3. <br />
      `scp YourOceanUsername@ocean:/PathToWaveTestDirectory/ScalarWave3DPeriodicVolume* .` 
      
6. Open Paraview and open ScalarWave3DPeriodicVolume.xmf. <br />
  - A window should appear that asks your what reader you want to open the data with. Select "XDMF Reader" because the other options will crash. <br />
  - Click "Apply" and change the Coloring option from "ErrorPhi_x" to Psi. <br />
  - Click play and we're done!
