
This page will assist you in submitting a Docker container to a challenge queue.
${toc}

---

## **1. Create a Synapse Project**
* Name your project `{challengeName} <Your team name>`
* You should also use this project for your final writeup submission, which should include your writeup as a Wiki, as well as prediction files (if any) and your source code in the **Files** tab.  View [**Writeup submission**](#!Synapse:syn18058986/wiki/594661) for more information. 
->${video?mp4SynapseId=syn20685869}<-
->_Creating a project on Synapse_<-

---


## **2. Build your Model**

This section will describe how to create your model and how it must take as parameters an input and output directory.

### Input files
* Your model should take as a parameter `--input`.
* All input files should be mounted in a directory called `/input` in the working directory of the container.
* If you have one input file, the convention is to name it `/input/input.csv`. 
* If you have multiple input files, these should be called `/input/input1.csv`, `/input/input2.csv` and so on.
* You should provide a description of what the input file(s) will look like in this section
* If you have multiple rounds, you should describe any differences between the input files for each round in this section.

### Output files
* All output files should be written into a directory called `/output` in the working directory of the container.
* If there is one output file, the convention is to name it `/output/predictions.csv`. 
* If there are multiple output files, these should be called `/output/predictions1.csv`, `/output/predictions2.csv` and so on.
* You should provide a description of what the output file(s) will look like in this section.
* If you have multiple rounds, you should describe any differences between the input files for each round in this section.

### Example
Here is an example of what an R script might look like, utilizing the conventions above:

_run_model.R_

```bash
run_model.R --input /input/directory --output /output/directory
```

```r
library(readr)
library(plyr)
library(optparse)

option_list = list(
  make_option("--input", type="character",
              help="Input data directory", metavar="character"),
  make_option("--output", type="character", default="/output", 
              help="Output directory [default= %default]", metavar="character")
) 
 
opt_parser = OptionParser(option_list=option_list);
opt = parse_args(opt_parser);

input_df <- read_csv(file.path(opt$input, "input.csv"))
model    <- readRDS("/usr/local/bin/model.rds")
scores   <- predict(model, input_df)
write_csv(output_df,  file.path(opt$output, "predictions.csv"))
```

---

## **3. Create a Dockerfile**

This section will describe how to write your Dockerfile. The Dockerfile describes the dependencies required to execute the Docker image.  These dependencies are encapsulated within the Docker image when it is built.  As such, the Docker image is a self-contained execution environment that will allow the Challenge organizers to run and reproduce your results.  This file must be named `Dockerfile`.

Here is an example Dockerfile using the **run_model.r** script created above:

_Dockerfile_
```
## Start from this Docker image
FROM ubuntu

## Install R in Docker image
RUN apt-get update && DEBIAN_FRONTEND=noninteractive apt-get install -y r-base

## Install R packages in Docker image
RUN Rscript -e 'source("http://bioconductor.org/biocLite.R"); biocLite("plyr")'
RUN echo "r <- getOption('repos'); r['CRAN'] <- 'http://cran.us.r-project.org'; options(repos = r);" > ~/.Rprofile
RUN Rscript -e "install.packages('readr')"

## Copy your files into Docker image
COPY run_model.R /usr/local/bin/
COPY model.rds /usr/local/bin/
RUN chmod a+x /usr/local/bin/run_model.R

## Make Docker container executable
ENTRYPOINT ["Rscript", "/usr/local/bin/run_model.R"]
```
The rest of this Wiki will go through each line in the **Dockerfile** example and explain its purpose.

### FROM (Pull from a base image)
The FROM command establishes what existing Docker image your image starts with. 
* Whenever possible, use current Official Repositories as the basis for your image.   
* We recommend using Ubuntu, e.g.
```
## Start from this Docker image
FROM ubuntu
```

### RUN (Install dependencies)
The most common use-case for RUN is an application of `apt-get` to install dependencies.  This example installs R, along with the R packages, plyr and readr:
```
## Install R in Docker image
RUN apt-get update && DEBIAN_FRONTEND=noninteractive apt-get install -y r-base

## Install R packages in Docker image
RUN Rscript -e 'source("http://bioconductor.org/biocLite.R"); biocLite("plyr")'
RUN echo "r <- getOption('repos'); r['CRAN'] <- 'http://cran.us.r-project.org'; options(repos = r);" > ~/.Rprofile
RUN Rscript -e "install.packages('readr')"
```

### COPY (Transfer local files into Docker image)
All files, including any scripts and their input files, should be copied into the Docker image; scripts should also be made executable. In this example, we are copying the script and model file from the previous section's example into the Docker image:
```
## Copy your files into Docker Container
COPY run_model.R /usr/local/bin/
COPY model.rds /usr/local/bin/
```
... as well as giving the script executable permissions:
```
RUN chmod a+x /usr/local/bin/run_model.R
```

### ENTRYPOINT (Make your Docker container executable)
The ENTRYPOINT  command specifies what gets executed when your Docker container is run. In this example, we want to run the Rscript we copied into `/usr/local/bin/`:
```
## Make Docker container executable
ENTRYPOINT ["Rscript", "/usr/local/bin/run_model.R"]
```

For more information, visit [**Best practices for writing Dockerfiles**](https://docs.docker.com/engine/userguide/eng-image/dockerfile_best-practices/).


---

## **4. Build a Docker image**

This section describes how to create your Docker image. You will need:
* Synapse ID of a project
* Dockerfile

### Set up your working directory.
* Move your Dockerfile and all files you are copying into your Dockerfile into the same directory.
* Make the above directory your current working directory.

### Build your Docker image
The basic syntax for creating a Docker image repository within your Synapse project is:
`docker build -t docker.synapse.org/<Your project ID>/<Repo name>:<Tag> <Dockerfile path>`
where:
* `<Your project ID>`: A Synapse project ID
* `<Repo name>`: The repository name will need to be unique in that namespace; it can be two to 255 characters, and can only contain lowercase letters, numbers or - and _.
* `<Tag>`: Optional.  If no tag is specified, a `latest` tag is added to your image. Tagging your image is very helpful, because it allows you to build different versions of your Docker image.
* `<Dockerfile path>`: Should be `.` since the Dockerfile should be in your current working directory.

In our example, we will use **my_model** as the repository name.  The Docker image repo may be created with a tag or without one, e.g.

```plaintext
# With tagging:
$ docker build -t  docker.synapse.org/syn12345/my-model:version1 .

# Without tagging:
$ docker build -t  docker.synapse.org/syn12345/my-model .
```

---

## **5. Upload your Docker image**

This section describes how to push your built Docker image from your local workstation up into Synapse. 

### Login to Synapse Docker Registry
Enter the following command, then answer its questions:

```bash
$ docker login docker.synapse.org
```

### View your built images (optional)

Before pushing your image, you can first check what you have built so far. This output shows the result of building **my-model** without tagging; notice that the TAG is `latest`. We have previously built several other models as well, which are listed:

```plaintext
$ docker images
REPOSITORY                                       TAG                 IMAGE ID            CREATED             SIZE
docker.synapse.org/syn12345/my-model           latest              e5993fdf4a41        8 minutes ago       736 MB
ubuntu                                        latest              14f60031763d        6 days ago          120 MB
docker.synapse.org/syn12345/expr-prolif-cyto   version1            50b113cf5444        13 days ago         962 MB
docker.synapse.org/syn12345/expr-cyto-srf      version2            0d88219f8319        3 weeks ago         972 MB
docker.synapse.org/syn12345/expr-cyto-srf      version1            1e8a50a8d345        3 weeks ago         972 MB
```

### Push your Docker image
You may now push your Docker image into Synapse, using the syntax:
`docker push docker.synapse.org/<Your project ID>/<Repo name>:<Tag>`

```
$ docker push docker.synapse.org/syn12345/my-model
```

Notice how a TAG is not included in the command above; recall that `latest` will be the default tag.  If there is a specific Docker image with a tag you want to push, e.g. **my-model:version1**, simply include the TAG name into the push command:
```
$ docker push docker.synapse.org/syn12345/my-model:version1
```

### Verify the Docker image was successfully pushed(optional)

If the Docker image was successfully pushed, it should show up in the **Docker** tab of your Synapse project page. You can navigate there by first going to your project page in Synapse:

`https://www.synapse.org/#!Synapse:<Your project ID>/docker` 

 The Docker image (e.g., `docker.synapse.org/syn12345/my-model`) should be listed. 

> We have occasionally experienced problems using older versions of Docker, where the previous push step appears to complete successfully, yet the image does not appear in the project's **Docker** tab. If you experience a similar issue, consider updating your version of Docker.


---

## **6. Submit your Docker image**

<!-- ${evalsubmit?projectId=syn18058986&unavailableMessage=Please register to make a submission.&buttonText=Submit to the challenge} -->

${image?fileName=docker%5Fsubmission%2Epng&align=Right&scale=50&responsive=true&altText=Submitting a Docker image}
To submit your Docker image, navigate to the Docker image uploaded on Synapse and click on the **Docker Repository Tools** button in the upper-right corner.  Select **Submit Docker Repository to Challenge** from the options, then follow the prompts. 



---


[< Back to: **How to Participate**](#!Synapse:syn18058986/wiki/588173)
