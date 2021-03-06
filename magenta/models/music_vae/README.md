# MusicVAE: A hierarchical recurrent variational autoencoder for music.

[MusicVAE](https://g.co/magenta/music-vae) learns a latent space of musical sequences, providing different modes
of interactive musical creation, including:

* random sampling from the prior distribution,
* interpolation between existing sequences,
* manipulation of existing sequences via attribute vectors or a [latent constraint model](https://goo.gl/STGMGx).

For short sequences (e.g., 2-bar "loops"), we use a bidirectional LSTM encoder
and LSTM decoder. For longer sequences, we use a novel hierarchical LSTM
decoder, which helps the model learn longer-term structures.

We also model the interdependencies among instruments by training multiple
decoders on the lowest-level embeddings of the hierarchical decoder.

Representations of melodies/bass-lines and drums are based on those used
by [MelodyRNN](/magenta/models/melody_rnn) and
[DrumsRNN](/magenta/models/drums_rnn).

For additional details, see our [blog post](https://g.co/magenta/music-vae) and [paper](https://goo.gl/magenta/musicvae-paper).

## How To Use

### Colab Notebook w/ Pre-trained Models

The easiest way to get started using a MusicVAE model is via our
[Colab Notebook](https://g.co/magenta/musicvae-colab).
The notebook contains instructions for sampling interpolating, and manipulating
musical sequences with pre-trained MusicVAEs for melodies, drums, and
three-piece "trios" (melody, bass, drums) of varying lengths.

### Generate script w/ Pre-trained Models

We provide a script in our [pip package](/README.md#installation) to generate
outputs from the command-line.

#### Pre-trained Checkpoints
Before you can generate outputs, you must either
[train your own model](#training-your-own-musicvae) or download pre-trained
checkpoints from the table below.

| ID | Config | Description | Link |
| -- | ------ | ----------- | ---- |
| mel_2bar | `cat-mel_2bar_big` | 2-bar melodies | [download](http://download.magenta.tensorflow.org/models/music_vae/checkpoints_bundled/mel_2bar_big.ckpt.tar)|
| mel_16bar | `hierdec-mel_16bar` | 16-bar melodies | [download](http://download.magenta.tensorflow.org/models/music_vae/checkpoints_bundled/mel_16bar_hierdec.ckpt.tar)|
| trio_16bar | `hierdec-trio_16bar` | 16-bar "trios" (drums, melody, and bass) | [download](http://download.magenta.tensorflow.org/models/music_vae/checkpoints_bundled/trio_16bar_hierdec.ckpt.tar)|
| drums_2bar_oh_lokl |`cat-drums_2bar_small` | 2-bar drums w/ 9 classes trained for more *realistic* sampling| [download](http://download.magenta.tensorflow.org/models/music_vae/checkpoints_bundled/drums_2bar_small.lokl.ckpt.tar)|
| drums_2bar_oh_hikl | `cat-drums_2bar_small` | 2-bar drums w/ 9 classes trained for *better reconstruction and interpolation* | [download](http://download.magenta.tensorflow.org/models/music_vae/checkpoints_bundled/drums_2bar_small.hikl.ckpt.tar)|
| drums_2bar_nade_full | `nade-drums_2bar_full` | 2-bar drums w/ 61 classes | [download](http://download.magenta.tensorflow.org/models/music_vae/checkpoints_bundled/drums_2bar_nade.full.ckpt.tar)|

Once you have selected a model, there are two operations you can perform with
the generate script: `sample` and `interpolate`.

#### Sample

Sampling decodes random points in the latent space of the chosen model and
outputs the resulting sequences in `output_dir`. Make sure you specify the
matching `config` for the model (see the table above).

Download the `mel_2bar_big` checkpoint above and run the following command to
generate 5 2-bar melody samples.

```sh
music_vae_generate \
--config=cat-mel_2bar_big \
--checkpoint_file=/path/to/music_vae/checkpoints/mel_2bar_big.ckpt \
--mode=sample \
--num_outputs=5 \
--output_dir=/tmp/music_vae/generated
```

Perhaps the most impressive samples come from the 16-bar trio model. Download
the `trio_16bar_hierdec` checkpoint above (warning: 2 GB) and run the following
command to generate 5 samples.

```sh
music_vae_generate \
--config=hierdec-trio_16bar \
--checkpoint_file=/path/to/music_vae/checkpoints/trio_16bar_hierdec.ckpt \
--mode=sample \
--num_outputs=5 \
--output_dir=/tmp/music_vae/generated
```

#### Interpolate

To interpolate, you need to have two MIDI files to inerpolate between. Each
model has ceratin constraints<sup id="a1">[*](#f1)</sup> for these files. For
example, the mel_2bar models only work if the input files are exactly 2-bars
long and contain monophonic non-drum sequences. The trio_16bar models require
16-bars with 3 instruments (based on program numbers): drums, piano or guitar,
and bass. `num_outputs` specifies how many points along the path connecting the
two inputs in latent space to decode, including the endpoints.

Try setting the inputs to be two of the samples you generated previously.

```sh
music_vae_generate \
--config=cat-mel_2bar_big \
--checkpoint_file=/path/to/music_vae/checkpoints/mel_2bar_big.ckpt \
--mode=interpolate \
--num_outputs=5 \
--input_midi_1=/path/to/input/1.mid
--input_midi_2=/path/to/input/2.mid
--output_dir=/tmp/music_vae/generated
```

<sup id="f1">*</sup>**Note**: If you call the generate script with MIDI files
that do not match the model's constraints, the script will try to extract any
subsequences from the MIDI files that would be valid inputs, and write them to
the output directory. You can then listen to the extracted subsequences, decide
which two you wish to use as the ends of your interpolation, and then call the
generate script again using these valid inputs. [↩](#a1)

### JavaScript w/ Pre-trained Models

We have also developed [MusicVAE.js](https://goo.gl/magenta/musicvae-js), a JavaScript API for interacting with
MusicVAE models in the browser. Existing applications built with this library include:

* [Beat Blender](https://g.co/beatblender) by [Google Creative Lab](https://github.com/googlecreativelab)
* [Melody Mixer](https://g.co/melodymixer) by [Google Creative Lab](https://github.com/googlecreativelab)
* [Latent Loops](https://goo.gl/magenta/latent-loops) by [Google Pie Shop](https://github.com/teampieshop)
* [Neural Drum Machine](https://codepen.io/teropa/pen/RMGxOQ) by [Tero Parviainen](https://github.com/teropa)

Learn more about the API in its [repo](https://goo.gl/magenta/musicvae-js).

### Training Your Own MusicVAE

If you'd like to train a model on your own data, you will first need to set up
your [Magenta environment](/README.md). Next, convert a collection of MIDI files
into NoteSequences following the instructions in
[Building your Dataset](/magenta/scripts/README.md). You can then choose one of
the pre-defined Configurations in [configs.py](configs.py) or define your own.
Finally, you must execute the [training script](train.py). Below is an example
command, training the `cat-mel_2bar_small` configuration and assuming your
examples are stored at `/tmp/music_vae/mel_train_examples.tfrecord`.

```sh
music_vae_train \
--config=cat-mel_2bar_small \
--run_dir=/tmp/music_vae/ \
--mode=train \
--examples_path=/tmp/music_vae/mel_train_examples.tfrecord
```

You will likely need to adjust some of the hyperparamters with the `--hparams`
flag for your particular train set and hardware. For example, if the default
batch size of a config is too large for your GPU, you can reduce the batch size
and learning rate by setting the flag as follows:

```sh
--hparams=batch_size=32,learning_rate=0.0005
```

These models are particularly sensitive to the `free_bits` and `max_beta`
hparams. Decreasing the effect of the KL loss (by increasing `free_bits` or
decreasing `max_beta`) results in a model that produces better reconstructions,
but with potentially worse random samples. Increasing the effect of the KL loss
typically results in the opposite. The default config settings of these hparams
are an attempt to reach a balance between good sampling and reconstruction,
but the best settings are dataset-dependent and will likely need to be adjusted.

Finally, you should also launch an evaluation job (using `--mode=eval` with a
heldout dataset) in order to compute metrics such as accuracy and to avoid
overfitting.

Once your model has trained sufficiently, you can load the checkpoint into the
[Colab Notebook](https://g.co/magenta/musicvae-colab) or use the
[command-line script](#pre-trained-checkpoints) to do inference and generate
MIDI outputs.
