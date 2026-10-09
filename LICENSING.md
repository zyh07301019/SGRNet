# SGRNet licensing and code provenance

SGRNet was further developed by Yuhua Zhang on the basis of [YorkQiu/PeopleRelationNetwork (PRN)](https://github.com/YorkQiu/PeopleRelationNetwork). It contains inherited framework code and the author's additional SGRNet development.

Adding substantial new code does not by itself change the ownership or licensing terms of inherited code. The author's own contributions and inherited PRN portions must be considered separately.

## Existing MIT file and its scope

The root [LICENSE](LICENSE) currently contains MIT text with copyright (c) 2026 Yuhua Zhang. That existing file has been retained unchanged during this documentation update.

An author can grant permission only for contributions they are entitled to license. This MIT file does not establish permission from the PRN copyright holders or authorize relicensing their inherited code. Applicable upstream permission and file-level scope still need confirmation before the entire repository is described as MIT-licensed.

## Is separate permission always necessary?

- If the applicable PRN version already has a license allowing modification and redistribution, comply with that license. It may already provide the necessary permission without a separate request.
- If you already have written permission, check that it covers modification, redistribution, and the intended license scope.
- If only the paper's ideas were used and no PRN code was copied or adapted, the code-reuse licensing question is different. That is not the provenance currently reported for this project.
- If inherited PRN code has no identifiable license or other permission, ask the rights holders to clarify the terms or grant the needed permission before redistributing that code.

On 2026-10-09, the complete public master tree at commit `5bb124b9d137543a31d9b6076d4bee22e7f169fd` was checked. No LICENSE, LICENCE, COPYING, or NOTICE file was found. This does not rule out permission provided elsewhere or for another version.

Attribution and a paper citation acknowledge the source; they do not replace permission to redistribute inherited code. See [GitHub's licensing guidance](https://docs.github.com/en/repositories/managing-your-repositorys-settings-and-features/customizing-your-repository/licensing-a-repository) and [GitHub's Open Source Guide](https://opensource.guide/legal/).

## Before public release

1. Record the original PRN version used and which files were inherited or modified.
2. Check existing permissions, file headers, and any license applicable to that version.
3. If no applicable permission is available, ask upstream rights holders to clarify or grant redistribution and modification permission.
4. Retain upstream notices and document the scope and compatible terms for SGRNet additions.

No message has been sent to upstream maintainers as part of this update.

## Dependencies, data, and weights

PyTorch, torchvision, NumPy, Pillow, PyYAML, SciPy, and tqdm have their own licenses. Dataset images and ImageNet-pretrained backbone weights have their respective terms. The SGRNet code license does not replace those terms.

SGRNet checkpoints are shared separately through Baidu Netdisk. The author-provided share link and extraction code are recorded in the README. Shared-file contents and downloads have not been independently verified during this documentation update.
