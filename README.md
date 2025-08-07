# TLS ClientHello Classifier

A machine learning-based TLS ClientHello packet classifier that leverages GraphSAGE for feature enrichment and employs a lightweight two-layer neural network for classification.

## Table of Contents

- [Overview](#overview)
- [Architecture](#architecture)  
- [Prerequisites](#prerequisites)
- [Installation](#installation)
- [Usage](#usage)
- [Methodology](#methodology)
- [Dataset](#dataset)
- [Results](#results)
- [Contributing](#contributing)
- [License](#license)
- [Contact](#contact)

## Overview

This project implements a TLS ClientHello packet classifier designed for high-speed network environments. The system combines graph neural networks (GraphSAGE) with traditional machine learning approaches to achieve accurate and efficient packet classification. The solution is optimized for real-time inference with minimal computational overhead.

## Architecture

The classification pipeline consists of three main components:

1. **Data Processing Module**: Extracts and preprocesses TLS ClientHello packets from network captures
2. **Feature Engineering Module**: Creates graph edges between features using cosine similarity and generates enriched embeddings via GraphSAGE
3. **Classification Module**: Performs packet classification using a lightweight two-layer neural network

![Architecture Diagram](proposed_architecture.jpg)

## Prerequisites

### Required Tools

- **Wireshark**: Network protocol analyzer for packet capture
  - Download from: [https://www.wireshark.org/](https://www.wireshark.org/)
  
- **TShark**: Command-line network protocol analyzer (typically bundled with Wireshark)
  - Verify installation: `tshark --version`
  
- **Netcap**: Network capture framework for packet processing
  - Visit: [https://netcap.io](https://netcap.io)
  - Used for filtering TLS ClientHello packets and JSON export

### Development Environment

- **Python 3.8+**
- **Code Editor**: VSCode, PyCharm, or similar IDE
- **Git**: For version control

## Installation

1. Clone the repository:
```bash
git clone https://github.com/alwyndsouza21/TLS-ClientHello-Classifier.git
cd TLS-ClientHello-Classifier
```

2. Create a virtual environment and install dependencies:
```bash
python -m venv venv       # venv : name of the virtual environment
source venv/bin/activate  # On Windows: venv\Scripts\activate

# Install required packages
pip install torch 
pip install torch-geometric
pip install torch-sparse
pip install scikit-learn pandas numpy matplotlib seaborn tqdm joblib
```

3. Verify tool installations:
```bash
wireshark --version
tshark --version
```

4. Download pre-trained models (if available) or prepare training data:
   - Place your CSV files in the appropriate directories as described in the Usage section

## Usage

### Data Collection

1. **Capture Network Traffic**:
   - Use Wireshark to capture packets while browsing target websites
   - Save captures as `.pcap` files

2. **Extract TLS ClientHello Packets**:
   ```bash
   netcap -r input.pcap -out output.json -filter tls
   ```

### Training and Classification

```bash
# Process the dataset
python preprocess_data.py --input data/raw --output data/processed

# Train the model
python train_model.py --config config/model_config.yaml

# Run inference
python classify.py --model models/classifier.pth --input test_data.pcap
```

## Methodology

### Feature Engineering

The system performs comprehensive feature analysis on TLS ClientHello packets:

- **Feature Selection**: Removes uncorrelated features through statistical analysis
- **Feature Engineering**: Derives new features, with `message_length` identified as a key discriminative feature
- **Data Cleaning**: Reduces noise and improves signal-to-noise ratio

### Graph Construction

Graph edges are constructed between features using:
- **Cosine Similarity**: Measures directional similarity between feature vectors
- **Top-K Selection**: Optimal k values of 5-10 for edge creation
- **Graph Structure**: Enables effective neighborhood sampling for GraphSAGE

### Classification Pipeline

1. **GraphSAGE Embedding**: Generates enriched feature representations through sampling and aggregation
2. **Neural Network**: Two-layer architecture optimized for speed and accuracy
3. **Inference Optimization**: Lightweight model suitable for high-throughput environments

For detailed technical background, refer to the [GraphSAGE paper](https://arxiv.org/abs/1706.02216).

## Dataset

The project uses custom-generated datasets from real network traffic:

**Data Sources:**
- `benign.csv`: TLS ClientHello packets from legitimate websites
- `malicious_12k.csv`: 12,000+ samples from malicious/suspicious sources
- Features extracted using tshark with specific TLS field filters

**TLS Fields Captured:**
```bash
tshark -Y "tls.handshake.type == 1 and !quic and tcp" -T fields \
  -e tls.record.length -e tls.handshake.cipher_suites_length \
  -e tls.handshake.extensions_length -e tls.handshake.ciphersuite \
  -e tls.handshake.sig_hash_alg -e tls.handshake.extensions_supported_group \
  # ... and more TLS-specific fields
```

**Data Preprocessing:**
- Balanced sampling between benign and malicious classes
- Feature engineering to create count-based metrics
- Normalization using StandardScaler
- Graph edge creation using cosine similarity

## Project Structure

```
TLS-ClientHello-Classifier/
├── Real_time_capture_1.py          # Real-time packet capture and classification
├── training_knn_csim.py             # Model training with k-NN graph construction
├── graphsage_model_without_TLS_version_k_value_*.pth  # Trained models (k=5,10,15,20,30)
├── scaler_without_tls_versions.pkl  # Feature scaler for normalization
├── proposed_architecture.jpg        # Architecture diagram
├── pcap_capture_benign/             # Benign training data directory
├── pcap_malicious_capture/          # Malicious training data directory
└── README.md                        # Project documentation
```

## Results

**Model Performance:**
- **Architecture**: GraphSAGE with 8→64→2 neurons
- **Training**: Multi-k value comparison (k=5,10,15,20,30)
- **Evaluation**: Comprehensive metrics including:
  - Confusion matrices with precision/recall
  - ROC-AUC curves for binary classification
  - Per-epoch training loss and accuracy tracking
- **Real-time Processing**: Batch processing of 10 packets with live classification results

**System Features:**
- **Multi-threaded Design**: Separate threads for packet capture and processing
- **Device Support**: Optimized for MPS (Apple Silicon) and CPU execution
- **Memory Efficient**: Neighbor sampling reduces computational complexity
- **Interpretable Output**: Domain-based classification results with SNI extraction

## Configuration

### Model Parameters

The system supports multiple configurations through different k-values for graph construction:

```python
# Available pre-trained models
k_values = [5, 10, 15, 20, 30]
model_files = [
    "graphsage_model_without_TLS_version_k_value_5.pth",
    "graphsage_model_without_TLS_version_k_value_10.pth",
    # ... etc
]
```

### Network Interface Configuration

Update the network interface in [`Real_time_capture_1.py`](Project/Real_time_capture_1.py):

```python
interface = "en0"  # macOS default
# interface = "eth0"    # Linux
# interface = "wlan0"   # WiFi
```

### Device Selection

The system automatically detects and uses the best available device:

```python
device = torch.device("mps" if torch.backends.mps.is_available() else "cpu")
```

## Troubleshooting

### Common Issues

1. **Permission Error with tshark**:
   ```bash
   sudo chmod +x /usr/bin/tshark
   # Or run with sudo permissions
   ```

2. **Network Interface Not Found**:
   - List available interfaces: `tshark -D`
   - Update the `interface` variable in the script

3. **Model Loading Issues**:
   - Ensure the correct model file exists
   - Check device compatibility (MPS vs CPU)

4. **Memory Issues**:
   - Reduce batch size in NeighborLoader
   - Lower k-value for graph construction

### Performance Optimization

- **For Real-time Processing**: Use k=5 or k=10 for faster inference
- **For Accuracy**: Use k=15 or k=20 for better classification performance
- **Memory Constrained**: Reduce `batch_size` and `hidden_channels`

## Quick Start

1. **Clone and Setup**:
   ```bash
   git clone <repository-url>
   cd TLS-ClientHello-Classifier
   pip install torch torch-geometric scikit-learn pandas numpy tqdm joblib
   ```

2. **Real-time Classification** (with pre-trained models):
   ```bash
   python Project/Real_time_capture_1.py
   ```

3. **Training from Scratch**:
   ```bash
   # Place your data files in the required directories
   python Project/training_knn_csim.py
   ```

## Contributing

1. Fork the repository
2. Create a feature branch (`git checkout -b feature/new-feature`)
3. Commit your changes (`git commit -am 'Add new feature'`)
4. Push to the branch (`git push origin feature/new-feature`)
5. Create a Pull Request

### Development Guidelines

- Follow PEP 8 coding standards
- Add comprehensive docstrings
- Include unit tests for new features
- Update documentation for API changes

## License

This project is licensed under the [LICENSE](LICENSE) file in the repository.

## Acknowledgments

- GraphSAGE implementation based on the original paper by Hamilton et al.
- TLS packet analysis inspired by network security research
- Real-time processing architecture designed for production environments

## Contact

**Alwyn D Souza**
Email: [alwyn14120@gmail.com](mailto:alwyn14120@gmail.com)

For questions, issues, or collaboration opportunities, please feel free to reach out or open an issue on GitHub.

---

*This project demonstrates the application of graph neural networks in network security and packet classification domains.*
