# ScalableLSH
This repository contains the following two components in our image search service:
- ./DiskE2LSH: Compute ITQ hashing results based on pre-computed pool5 features.
- ./Deploy: Deploy the server to receive a query image, compute features and perform image search

## Requirements ##
This code is written in C++ and requires the following libraries:
- LevelDB
- Boost
- Eigen

## Compile ##
- Run the Makefile stored in ./DiskE2LSH.
- Run the Makefile stored in ./Deploy.

## Usage ##
We first compute ITQ hashing results:
```
./buildIndex.bin \
    -d <datapath> "Path to LMDB where the data is store" \
    -n <imgslist> "Filenames of all images in the corpus"  \
    -s <save> "Path to save the hash table" \
    -b <nbits> "Number of bits in the representation" \
    -t <ntables> "Number of random proj tables in the representation" \
    -a <saveafter> "Time after which to snapshot the model (seconds)" \
    --nTrain "Number of random elements from imgComputeIDs to be used to train ITQ"
```

Then we deploy the server:
```
./computeFeatAndSearch.bin \
    -n <network-path> "Path to corresponding caffemodel" \
    -m <model-path> "Path to corresponding caffemodel" \
    -l <layer> "CNN layer to extract features from" \
    -i <index> "Path to load search index from" \
    -s <featstor> "Path to feature store" \
    --imgslist "File with images list"  \
    --port-num "Port to run the service on" \
    --num-output "Max number of matches to return" \
    --nRerank "Max number of images to re-rank using actual features " \
    -f <foregorund> "Extract the features of the foreground (1) or background (0)"
```

