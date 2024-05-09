---
Title: "Running Tratteria"
weight: 1
toc: true
---

This quick-start guide outlines the steps to set up and run Tratteria for initial learning and evaluation purposes.

&nbsp;

- [Getting the Repository](#getting-the-repository)

- [Quick-Start Configuration](#quick-start-configuration)

- [Running as a Native Go Server](#running-as-a-native-go-server)

- [Running as a Docker Container](#running-as-a-docker-container)

&nbsp;
### Getting the Repository

To get started, clone the Tratteria repository from GitHub:

```bash
git clone https://github.com/SGNL-ai/Tratteria
```

&nbsp;
### Quick Start Configuration

Tratteria is configured using a YAML file. The quick-start configuration can be found at:

[https://github.com/SGNL-ai/tratteria/blob/main/example-configs/config.quick-start.yaml](https://github.com/SGNL-ai/tratteria/blob/main/example-configs/config.quick-start.yaml)

The file contains the below configuration:

```yaml
issuer: https://example.org/tts
audience: https://example.org/
token:
  lifeTime: "15s"
subjectTokens:
  selfSigned:
    validation: false
enableAccessEvaluation: false
```

##### Explanation of Configuration Parameters:

- **issuer**: Specifies the txn-token server that issues the token.
- **audience**: Intended recipient(s) of the token.
- **token**: Configuration settings for tokens:
  - **lifeTime**: Duration for which the token is valid; here, it is 15 seconds.
- **subjectTokens**: Configuration settings for subject tokens, which are unique identifiers for the subjects making the requests:
  - **selfSigned**: Configuration for self-signed JWT subject tokens.
    - **validation**: Wheather validation is enabled for self-signed tokens; set to false in this basic setup.
- **enableAccessEvaluation**: Flag to enable or disable access evaluation, disabled here to simplify the setup.

This configuration represents the most minimal settings necessary to get Tratteria running.

&nbsp;
### Running as a Native Go Server

##### Prerequisites

You need to have Go installed on your system. For running Tratteria, Go version 1.22.0 is recommended. You can download it from the [official Go website](https://golang.org/dl/).

##### Setting Up the Configuration

Once Go is installed, you can set up your service configuration. Copy the quick-start configuration into the service repository:

```bash
cp example-configs/config.quick-start.yaml service/config.quick-start.yaml
```

##### Running Tratteria

Change the directory to the service folder:

```bash
cd service
```

To run the service, you have two options:

**Direct Execution:**

You can run the service directly using the Go command. This method compiles and runs the program in one step:

```bash
go run cmd/main.go config.quick-start.yaml
```

**Compile and Run:**

Alternatively, compile the service into an executable:

```bash
go build -o tratteria ./cmd
```

Then, run the executable:

```bash
./tratteria config.quick-start.yaml
```

##### Accessing Tratteria

After starting the service, it will be available at `localhost:9090`.


&nbsp;
### Running as a Docker Container

##### Setting Up the Configuration

Copy the quick-start configuration into the service repository:

```bash
cp example-configs/config.quick-start.yaml service/config.quick-start.yaml
```

##### Building the Docker Image

First, change to the service directory:

```bash
cd service
```

Build the Docker image:

```bash
docker build -t tratteria:latest .
```

##### Running the Docker Image

Run the Docker container:
```bash
docker run \
  -v $(pwd)/config.quick-start.yaml:/app/config.yaml \
  -p 9090:9090 \
  tratteria:latest /app/config.yaml
```

##### Accessing Tratteria

This starts the Tratteria server inside a Docker container and makes it accessible at `localhost:9090`.



&nbsp;

After successfully running the Tratteria, you might want to generate txn-tokens (TraTs). For detailed instructions on this next step, please visit [Generating Txn-Tokens (TraTs)](/docs/quickstart/generating-trats).

This quick-start guide provides the basic steps to get Tratteria up and running for initial learning and evaluation purposes. Remember to adjust the configurations as per the requirements for any setup beyond initial evaluations. Check [this guide](#) for detailed instructions on configuring and integrating production-level Tratteria.