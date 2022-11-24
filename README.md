# LanguageTool

> [LanguageTool](https://www.languagetool.org/) is an Open Source proofreading software for English, French, German, Polish, Russian, and [more than 20 other languages](https://languagetool.org/languages/). It finds many errors that a simple spell checker cannot detect.

The source repo can be found [here](https://github.com/meyayl/docker-languagetool).

About this image:
- Uses official release zip from https://languagetool.org/download/
- Uses the latest Alpine 3.16 base image
- Uses OpenJDK 17
- includes fasttext
- includes su-exec
  - container starts as root and executes languagetool as restricted user using `exec su-exec`
  - container fixes folder ownership for ngrams and fasttext folders
- optional: downloads ngram language modules if configured (if they don't already exist)
- optional: downloads fasttext module (if it doesn't already exist)
- optional: user mapping (make sure to check MAP_UID and MAP_GID below)

Note: due to proper pid1 handling, the container will exit with status code 143 (=SIGTERM). It appears that languagetool does not handle SIGTERM, as such even though the container is terminated the way it should be, it will show the status code.

# Setup

## Docker CLI Usage

```sh
docker run -d \
 --name=languagetool \
 --env download_ngrams_for_langs=en \
 --env langtool_languageModel=/ngrams \
 --env langtool_fasttextModel=/fasttext/lid.176.bin \
 --volume $PWD/ngrams:/ngrams \
 --volume $PWD/fasttext:/fasttext \
  meyay/languagetool:latest
```

## Docker Compose Usage

```
---
version: "3.8"

services:
  languagetool:
    image: meyay/languagetool:latest
    container_name: languagetool
    ports:
      - 8010:8010
    environment:
      download_ngrams_for_langs: en
      langtool_languageModel: /ngrams
      langtool_fasttextModel: /fasttext/lid.176.bin
    volumes:
      - ./ngrams:/ngrams
      - ./fasttext:/fasttext
```

## Parameters

The environment parameters are split into two halves, separated by an equal, the left-hand side represents the variable names (use them as is) the right-hand side the value (change if necessary).

| ENV| DEFAULT | DESCRIPTION |
| ------ | ------ | ------ |
| download_ngrams_for_langs | none | Optional: Comma separated list of languages to download ngrams for. Skips download if the ngrams for that language already exist. Valid languages: `en`, `de`, `es`, `fr` and `nl`. Example value: `en,de` |
| langtool_languageModel | /ngrams | Optional: The base path to the ngrams models. |
| langtool_fasttextBinary | /usr/local/bin/fasttext | Optional: Path to the fasttext binary. Change only if you want to test your own compiled binary. Don't forget to map it into the container as volume. |
| langtool_fasttextModel |  | Optional: The container path to the fasttext model binary. If the variable is set, the fasttext model will be downloaded if doesn't exist yet. |
| langtool_*|  |  Optional: An arbitrary languagetool configuration, consisting of the prefix `langtool_` and the key name as written in the property file. |
| JAVA_XMS | 256m | Optional: Minimum size of the Java heap space. Valid suffixes are `m` for megabytes and `g` for gigabytes.|
| JAVA_XMX | 1024m | Optional: Maximum size of the Java heap space. Valid suffixes are `m` for megabytes and `g` for gigabytes. Set a higher value if you experience OOM kills, but do not use more than 1/4 of the host memory! |
| JAVA_GC | SerialGC | Optional: Configure the garbage collector the JVM will use. Valid options are: `SerialGC`, `ParallelGC`, `ParNewGC`, `G1GC`, `ZGC` |
| JAVA_OPTS | | Optional: Set you own custom Java options for the JVM. This will render the other JAVA_* options useless. |
| MAP_UID | 783 | Optional: UID of the user inside the container that runs LanguageTool. If you encounter permission problems with your volumes, make sure to set the parameter to the UID of the host folder owner. |
| MAP_GID | 783 | Optional: GID of the user inside the container that runs LanguageTool. If you encounter permission problems with your volumes, make sure to set the parameter to the GID of the host folder owner. |

# Change Log

| Date | Tag | Change |
|---|---|---|
| 2022-11-24 | 5.9-3 | - Add support to configure garbage collector </br> - Add JVM argument `-XX:+UseStringDeduplication` </br> - Add support to pass custom JAVA_OPTS </br> - Change Java_Xm? variables to JAVA_XM? |
| 2022-11-12 | 5.9-2 | - Update base image to Alpine 3.16.3 |
| 2022-09-28 | 5.9-1 | - Update LanguageTool to 5.9 |
| 2022-09-10 | 5.8-2 | - Add user mapping support |
| 2022-09-10 | 5.8-1 | - Initial release with Alpine 3.16.2, LanguageTool 5.8 |