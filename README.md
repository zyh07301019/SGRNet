# SGRNet

PyTorch implementation of **Scale-aware Gated Relation Network for Important Person Detection**.

**Paper authors:** Yuhua Zhang, Haifeng Sang, and Qing Liu  
**Code author:** Yuhua Zhang  
**Affiliation:** Shenyang University of Technology

SGRNet was further developed by Yuhua Zhang on the basis of the [People Relation Network (PRN)](https://github.com/YorkQiu/PeopleRelationNetwork) implementation. It retains parts of the PRN framework and adds the author's SGRNet components:

- **Gaussian-Guided Spatial Prior Modeling (GSPM):** a continuous spatial prior based on a candidate bounding box center and dimensions.
- **Scale-aware Gated Relation Modeling (SGRM):** directed relative area information and a target-conditioned gate for person relations.
- **Adaptive Normalized Fusion (ANF):** L2-normalized branch features with learnable Softmax fusion weights.

The task takes an image and dataset-provided candidate bounding boxes as input and ranks the candidates by their importance scores. This repository does not provide a separate person detection model.

## Release status

The model class, superclass initialization, and both configuration files now consistently use `SGRNet`. The previous class-name mismatch has been resolved. Read [RELEASE_CHECKLIST.md](RELEASE_CHECKLIST.md) for remaining release and evaluation checks.

The root [LICENSE](LICENSE) currently contains MIT text with copyright (c) 2026 Yuhua Zhang. It does not establish permission to redistribute or relicense inherited PRN code. Upstream permission and the final license scope need confirmation; see [LICENSING.md](LICENSING.md) and [THIRD_PARTY_NOTICES.md](THIRD_PARTY_NOTICES.md).

### Retained implementation details

The author has chosen to retain the current spatial-map generation and geometric center calculation:

- Training random crops use binary spatial maps; evaluation uses Gaussian spatial maps.
- Geometric edges use the current center expression `(x + w) // 2, (y + h) // 2`, which differs from the conventional xywh center `x + w / 2, y + h / 2`.

These details are documented for reproducibility. They have not been changed during documentation preparation.

## Reported results

The manuscript reports the following test-set results. They have **not been independently re-evaluated during this repository packaging pass**.

| Dataset | mAP (%) | Checkpoint selection |
| --- | ---: | --- |
| MS | 94.14 | Final training epoch, 400 |
| NCAA | 98.36 | Final training epoch, 200 |

For each image, candidates are sorted by positive-class probability. The current evaluator uses trapezoidal integration of the precision-recall curve and averages AP over images. The printed mAP is a fraction; multiply by 100 to express it as a percentage.

## Repository structure

```text
SGRNet/
├── main.py                         # Training, evaluation, and resume entry point
├── preprocess_datasets.py          # Dataset annotation and image preprocessing
├── configs/
│   ├── bestMS.yaml
│   └── bestNCAA.yaml
├── packages/
│   ├── models/
│   │   ├── SGRNetwork.py
│   │   └── blocks/
│   └── utils/
│       ├── data/
│       ├── init.py
│       ├── train_eval_test.py
│       ├── tools.py
│       └── ckpt.py
├── best_model/                      # Local checkpoints; excluded from Git
├── requirements.txt
├── .gitignore
├── LICENSE                         # MIT text; scope discussed in LICENSING.md
├── LICENSING.md
├── THIRD_PARTY_NOTICES.md
└── RELEASE_CHECKLIST.md
```

## Environment and installation

The author supplied the following environment versions:

| Component | Version |
| --- | --- |
| Python | 3.9.21 |
| PyTorch | 2.5.1 |
| torchvision | 0.20.1 |
| PyTorch CUDA runtime | 11.8 |
| NumPy | 2.0.2 |
| Pillow | 11.1.0 |
| PyYAML | 6.0.2 |
| SciPy | 1.13.1 |
| tqdm | 4.67.1 |

The manuscript reports Ubuntu 24.04 and an NVIDIA GeForce RTX 3080 Ti. The current entry point requires Linux/WSL and CUDA because it directly uses the Unix `resource` module and CUDA tensors.

Create an environment:

```bash
conda create -n sgrnet -c conda-forge python=3.9.21 pip
conda activate sgrnet
```

Install the matched CUDA 11.8 PyTorch packages first, then the remaining requirements:

```bash
python -m pip install torch==2.5.1 torchvision==0.20.1 --index-url https://download.pytorch.org/whl/cu118
python -m pip install -r requirements.txt
python -m pip check
```

This pair and CUDA wheel index are listed in the [official PyTorch installation history](https://pytorch.org/get-started/previous-versions/). The requirements file pins the seven directly imported third-party packages, rather than the complete transitive environment.

Check the installed versions and GPU availability:

```bash
python -c "import sys, torch, torchvision; print('Python:', sys.version); print('torch:', torch.__version__); print('torchvision:', torchvision.__version__); print('CUDA:', torch.version.cuda); print('GPU available:', torch.cuda.is_available())"
```

The model initializes three ImageNet-pretrained ResNet-50 backbones through `ResNet50_Weights.DEFAULT`. Initial construction may download pretrained weights. Record the resolved weights enum and keep the torchvision version fixed for reproduction.

## Data preparation

Download resources for MS and NCAA are listed in the [upstream PRN README](https://github.com/YorkQiu/PeopleRelationNetwork#datasets). Dataset use remains subject to the respective dataset terms.

The expected directory structure is:

```text
datasets/
├── MS/
│   ├── data/
│   │   └── annotations.mat
│   └── images/
│       ├── train/
│       ├── val/
│       └── test/
└── NCAA/
    ├── data/
    │   └── annotations.mat
    └── Images/
        ├── train/
        ├── val/
        └── test/
```

The uppercase `Images` for NCAA matches the current script. Use the partitions supplied with the dataset.

Before preprocessing, edit `set_paths()` in `preprocess_datasets.py` to point to your annotations, image directories, and output directories. Create the output directories if needed.

Run from the project root:

```bash
python preprocess_datasets.py --dataset MS --need_multi_vip
python preprocess_datasets.py --dataset NCAA --need_multi_vip
```

The flag `--need_multi_vip` generates `processed(multi).pkl` and `processed(multi).json`, matching the example configurations. Omitting the flag keeps only images with one positive candidate and writes differently named files.

Preprocessing removes invalid boxes and images without positive candidates. Record the number of retained images in each split and compare it with the experiment data. The generated files currently contain full image paths; moving the dataset requires regenerating them or updating those paths.

In both YAML files, set:

```yaml
models_path: /absolute/path/to/SGRNet/packages/models
data:
  path:
    MS: /absolute/path/to/SGRNet/datasets/MS/data/processed(multi).pkl
    NCAA: /absolute/path/to/SGRNet/datasets/NCAA/data/processed(multi).pkl
```

Update the existing `data.path` entries while retaining `data.mean`, `data.std`, and `data.data_augment`. The snippet above is not a complete replacement configuration.

Both supplied configurations use `model.type: SGRNet`, matching the current `SGRNet` Python class. Preserve that match when creating additional configurations.

## Training

After preparing paths and data:

```bash
python main.py -G 0 -C configs/bestMS.yaml
python main.py -G 0 -C configs/bestNCAA.yaml
```

Use a single visible GPU initially. Always pass `-G 0` or an appropriate GPU ID; the current default is `-1`, while the model still calls CUDA.

| Setting | MS | NCAA |
| --- | ---: | ---: |
| Epochs | 400 | 200 |
| Image batch size | 3 | 3 |
| Candidates per training image | 8 | 8 |
| Random seed | 4 | 3 |
| Initial learning rate | 0.001 | 0.001 |
| StepLR interval | 50 | 25 |
| StepLR multiplier | 0.5 | 0.5 |

Both configurations use SGD with momentum 0.9 and weight decay 0.0005. Final checkpoints are written to `saves/MS/BestMS.pkl` and `saves/NCAA/BestNCAA.pkl`. Training records are written under `records/`.

An existing record file stops a fresh training run. The `--force` option replaces that record file; use a separate model name/configuration to retain earlier experiments.

## Evaluation

The trained checkpoints are shared through **Baidu Netdisk**, in the author-provided shared folder named **model**. They are not included in the Git repository. Both datasets use the same share link and extraction code.

| Dataset | Checkpoint filename | Baidu Netdisk share link | Extraction code |
| --- | --- | --- | --- |
| MS | `BestMS.pkl` | [Baidu Netdisk](https://pan.baidu.com/s/1TR64nDDMyVTJGputRzJ-LA?pwd=rj4z) | `rj4z` |
| NCAA | `BestNCAA.pkl` | [Baidu Netdisk](https://pan.baidu.com/s/1TR64nDDMyVTJGputRzJ-LA?pwd=rj4z) | `rj4z` |

After download, place the files at `best_model/BestMS.pkl` and `best_model/BestNCAA.pkl`. Keep these filenames to use the evaluation commands below.

Each local checkpoint is approximately 0.8 GB. The share link was supplied by the author; this documentation update has not independently verified the shared folder contents or downloaded files. Verify the downloaded filenames and checkpoint compatibility before evaluation.

Run from the project root on Linux/WSL. The absolute paths below allow the existing loader to use the `best_model` directory:

```bash
python main.py -G 0 -C configs/bestMS.yaml --eval -RM "$PWD/best_model/BestMS.pkl"
python main.py -G 0 -C configs/bestNCAA.yaml --eval -RM "$PWD/best_model/BestNCAA.pkl"
```

The loader currently expects a training-checkpoint dictionary containing `epoch`, `state_dict`, `optimizer`, and `scheduler`. Check compatibility with the supplied files. This packaging pass verified their presence and size, but did not deserialize or evaluate them.

## Resume training

For an existing checkpoint produced by this code, use a command such as:

```bash
python main.py -G 0 -C configs/bestMS.yaml -r -RM "$PWD/saves/MS/BestMS_50_.pkl"
```

Substitute a checkpoint that actually exists. The scheduler save/resume timing requires correction before claiming equivalence to uninterrupted training; see the release checklist.

## Implementation map

| Component | Main implementation |
| --- | --- |
| GSPM, evaluation path | `BaseDataset.crop_box()` in `packages/utils/data/dataset.py` |
| Spatial prior, training-crop path | `VIPRandomResizedCrop.get_face_and_ctx_by_bbox()` in `packages/utils/data/transforms.py`; retained binary maps |
| SGRM geometric edges | `SGRNet.get_edges()` in `packages/models/SGRNetwork.py` |
| SGRM relation gate | `InterPersonAttention.forward()` in `packages/models/blocks/attention.py` |
| ANF | `SGRNet.forward()` in `packages/models/SGRNetwork.py` |
| Per-image AP and mAP | `get_mAP()` in `packages/utils/tools.py` |

## Citation and acknowledgments

SGRNet is a further development of the PRN implementation, with additional spatial-prior, relation-modeling, and fusion components. Acknowledge the upstream framework and cite its paper when using the inherited implementation:

```bibtex
@article{Qiu2022PRN,
  title={Learning Relation Models to Detect Important People in Still Images},
  author={Qiu, Yu-Kun and Hong, Fa-Ting and Li, Wei-Hong and Zheng, Wei-Shi},
  journal={IEEE Transactions on Multimedia},
  year={2022},
  publisher={IEEE}
}
```

For SGRNet, the supplied manuscript can be cited provisionally as:

```bibtex
@unpublished{zhang_sgrnet,
  title={Scale-aware Gated Relation Network for Important Person Detection},
  author={Zhang, Yuhua and Sang, Haifeng and Liu, Qing},
  note={Manuscript}
}
```

Add the verified publication year, venue, DOI, paper URL, and repository URL when available.

## Licensing

SGRNet includes inherited PRN code and the author's additional development. The existing [MIT text](LICENSE) does not establish licensing authority over the inherited parts. Confirm applicable upstream permission and document the file-level scope before presenting the entire repository as MIT-licensed.

Third-party dependencies, dataset images, and pretrained-backbone weights remain subject to their respective terms. See [LICENSING.md](LICENSING.md) and [THIRD_PARTY_NOTICES.md](THIRD_PARTY_NOTICES.md).
