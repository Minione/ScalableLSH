# ScalableLSH
This repository contains the following two components in Memex image search service:
    
    1. ./DiskE2LSH: Compute ITQ hashing results based on pre-computed pool5 features.
    2. ./Deploy: Deploy the server to receive a query image, compute features and perform image search

DiskE2LSH takes the features of database images pre-computed by [ComputeFeatures](https://github.com/Minione/ComputeFeatures) as input and apply [Iterative Quantization (ITQ)](http://slazebni.cs.illinois.edu/publications/cvpr11_small_code.pdf) to map these high-dimensional float features to low-dimension hash codes or binary codes.

Deploy is used to set up the image search service that the user can access to perform image search. When the service receives one query image, it first extract float features for the query from a pre-trained nerual network (e.g., pool from AlexNet) and then applies ITQ to map the float query feature to hash codes. Then we use Hamming distance to measure the distance between the query and database images and return an initial ranked list of retrieved images (contains 'num-output' images). At the end, we re-rank the top 'nRerank' returned images with the original float features, where Euclidean distance is employed. 'num-output' and 'nRerank' are two parameters, specified when lauching the service. In Memex, we deploy image search services on [Counterfeit Electronics Dataset](https://memexproxy.com/wiki/display/MPM/CMU%27s+Image+Search+Services+for+Counterfeit+Electronics+Dataset), [Weapons Dataset](https://memexproxy.com/wiki/display/MPM/CMU%27s+Image+Search+Services+for+Weapons+Dataset) and [HT Dataset](https://memexproxy.com/wiki/pages/viewpage.action?pageId=8062117).

## Requirements ##
This code is written in C++ and requires the following libraries:
- OpenCV3
- LevelDB
- Boost
- Eigen
- zeromq

## Compile ##
- Run the Makefile stored in ./DiskE2LSH.
- Run the Makefile stored in ./Deploy.

Note that some libraries are not installed in the default path in our environment, so we include the library path for zeromq, boost and OpenCV3 in the example Makefile explicitly. You will need to manually modify the Makefile with correct library paths in your own environment to successfully compile the code.

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

