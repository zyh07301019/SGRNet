# Third-party sources, components, and resources

## People Relation Network

SGRNet was further developed by Yuhua Zhang on the basis of the PRN implementation, retaining parts of its framework and adding SGRNet-specific components.

- Repository: https://github.com/YorkQiu/PeopleRelationNetwork
- Paper: Learning Relation Models to Detect Important People in Still Images
- Upstream authors: Yu-Kun Qiu, Fa-Ting Hong, Wei-Hong Li, and Wei-Shi Zheng
- Reported venue: IEEE Transactions on Multimedia
- Inspected public master commit: `5bb124b9d137543a31d9b6076d4bee22e7f169fd`
- Inspection date: 2026-10-09
- Permission status: no license file was found in that complete tree; applicable permission needs confirmation.

The inspected commit is not asserted to be the historical checkout used for SGRNet. Record that original version and the inherited/modified files before release.

The SGRNet additions described in the manuscript include spatial-prior modeling, scale-aware gated relations, and adaptive normalized fusion. Original contributions do not replace the rights or required notices for inherited PRN code.

## Runtime dependencies

The project imports PyTorch, torchvision, NumPy, Pillow, PyYAML, SciPy, and tqdm. These packages have their own licenses, are installed separately, and are not bundled with this source release. Refer to their official distributions for license texts and copyright notices.

## Pretrained backbones and datasets

The model initializes ImageNet-pretrained ResNet-50 backbones through torchvision. Those weights and the MS/NCAA dataset images remain subject to their respective terms.

SGRNet checkpoints are shared separately through Baidu Netdisk; the author-provided share link and extraction code are recorded in the README.

## License boundary

The existing root MIT text does not establish permission to redistribute or relicense the inherited PRN portions. See [LICENSING.md](LICENSING.md) for pending permission and scope checks. This notice records provenance and does not itself grant upstream permission.
