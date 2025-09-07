# CMK Go
Play Go/Weiqi/Baduk with a Neural Net in a web browser<br>

# Supported browsers
 - Chrome mobile/desktop
 - Firefox desktop
<br>
<a href="https://maksimkorzh.github.io/cmkgo/">Play now against FOX 1 Dan net</a><br><br>
Firefox mobile is working only with CPU backend because its WebGL
has issues with floating point precision and calculates forward pass
in a completely misleading way.

# Thanks for testing
 - <strong>wurf3</strong> from OGS for reporting Firefox Mobile bug
 - <strong>ngenvi</strong> from OGS for questioning how EVAL works

# GTP mode
1. Download source code
2. Download neural net from release page
3. Create folder "model" under /cmkgo/train/
4. Copy/paste "model.json" and "weights.bin" to "model" folder
5. Connect "gtp.js" to GoGUI or other user interface

# Training results
I trained three policy nets (no value heads) on Intel Core i5-10400 CPU @ 2.90GHz × 6<br>
using ~1M training positions from Pro games (years 2013-2017) and ~1.5K FOX Pro games:
<li><strong>cmkgo-cnn11c96-s1067091</strong></li>
<li><strong>cmkgo-b6c96-s1067091</strong></li>
<li><strong>cmkgo-b10c128-s1119000</strong></li>
<br>
Following table shows training results comparison:
<br>
<br>
<table>
  <tr>
    <th>Name</th>
    <th>Arch</th>
    <th>Size</th>
    <th>Time</th>
    <th>Loss</th>
    <th>Acc. true</th>
    <th>Acc. pred</th>
    <th>Strength</th>
  </tr>
  <tr>
    <td>cnn11c96</td>
    <td>11 layer CNN with dense output layer, 96 convolutional filters</td>
    <td>~56Mb</td>
    <td>~28hr</td>
    <td>2.14</td>
    <td>39.15% (~20k samples)</td>
    <td>32.05% (~17k samples)</td>
    <td>~ FOX 10 kyu</td>
  </tr>
  <tr>
    <td>b6c96</td>
    <td>6 residual blocks, 96 convolutional filters (katago style)</td>
    <td>~5Mb</td>
    <td>~52hr</td>
    <td>2.21</td>
    <td>41.37%</td>
    <td>37.75%</td>
    <td>~ FOX 6 kyu</td>
  </tr>
  <tr>
    <td>b10c128</td>
    <td>10 residual blocks, 128 convolutional filters (katago style)</td>
    <td>~12Mb</td>
    <td>~75hr</td>
    <td>2.07</td>
    <td>46.26%</td>
    <td>38.04%</td>
    <td>~ FOX 1 dan</td>
  </tr>
</table>

# How to train your own net
    # Installation
    git clone https://github.com/maksimKorzh/cmkgo
    cd cmkgo/train
    
    # x86_64 systems
    npm install
    
    # For raspberry pi 5 (mostly for running "gtp.js"):
    npm install @tensorflow/tfjs-node@4.8.0
    npm rebuild @tensorflow/tfjs-node --build-from-source

    ./download.sh              # downloads games from https://badukmovies.com
    python extract_games.py    # extract moves from SGFs, write them to "games.js"
    node build_dataset.js      # creates X.bin and Y.bin training data files
    node train.js              # train neural net (you may want to adjust params or model arch)
    node gtp.js                # used to play against the net in GoGUI, make sure "path/to/model" is correct
    ./pack-model.sh            # Make model 75% smaller (optional)
    
    accuracy.js                # used to evaluate NN accuracy in percents,
                               # assumes "model.json", "weights.bin",
                               # "X_train.bin", "Y_train.bin", "X_val.bin", "Y_val.bin"
                               # to be in "./test" folder
    
    playok.js                  # plays vs human players at "playok.com/go" under guest account

    NOTE: you may run out of RAM if processing too many games at once,
          so the suggested way is to extract games year by year (alter
          extract_games.py) and then run "build_dataset.js" to append
          newly encoded games to "./dataset/X.bin" and "./dataset/Y.bin".
          Alternatively you can populate "games.js" with your own games
          all at once or in stages - "build_dataset.js" would append
          encoded moves from listed games to *.bin files. Make sure
          you keep track of eventual number of positions and put this
          number into totalSamples variable in "train.js" to have a
          proper current/total samples rate.
