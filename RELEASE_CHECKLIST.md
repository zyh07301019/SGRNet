# SGRNet 发布说明和最终检查

更新日期：2026-10-09。以下行号针对本次读取的代码；后续修改可能改变行号。

代码按作者的最新说明记录为：Yuhua Zhang 在 PRN 实现基础上进一步开发 SGRNet，保留部分框架并加入自己的模块。高斯图和中心公式按作者决定保持当前实现。本次只修正文档，未修改 Python 源码、YAML 配置、权重或现有 LICENSE。

## 模型类名已统一

当前源码和配置已经统一使用 SGRNet：

| 位置 | 当前内容 |
| --- | --- |
| configs/bestMS.yaml 第 3 行 | type: SGRNet |
| configs/bestNCAA.yaml 第 3 行 | type: SGRNet |
| packages/models/SGRNetwork.py 第 13 行 | class SGRNet(nn.Module): |
| packages/models/SGRNetwork.py 第 15 行 | super(SGRNet, self).__init__() |
| packages/utils/init.py 的 init_model()，第 57 行 | 按类名精确匹配，现在两端名称一致 |

此前类名不匹配已经修复。本次仅确认作者更新后的源码并同步说明，没有再次改动源码或配置。

## 按作者决定保留的实现

| 项目 | 位置 | 当前实现 |
| --- | --- | --- |
| 训练空间图 | packages/utils/data/transforms.py，VIPRandomResizedCrop.get_face_and_ctx_by_bbox()，第 203–215 行 | 生成二值图，取值 0/255 |
| 评估空间图 | packages/utils/data/dataset.py，BaseDataset.crop_box()，第 261–274 行 | 生成高斯图 |
| 几何中心公式 | packages/models/SGRNetwork.py，SGRNet.get_edges() 内 get_center()，第 150–153 行 | 保持 (x+w)//2、(y+h)//2 |

这些实现保留现状，不再作为本次待修改任务。保留不表示它们与常规 xywh 中心定义相同，也不表示本次已验证论文指标。

## 其他运行注意事项

| 项目 | 位置 | 说明 |
| --- | --- | --- |
| 权重不存在时异常不清晰 | packages/utils/ckpt.py，resume_model()，第 46–59 行 | start_epoch 只在找到文件时赋值；先确认权重路径真实存在 |
| 恢复训练的调度器时序 | main.py 第 108、123 行；ckpt.py 第 34、51 行 | 保存发生在 scheduler.step 前，但恢复从下一轮开始；恢复训练前核对学习率序列 |
| 权重目录 | ckpt.py 第 47 行；init.py 第 183–185 行 | README 用绝对 -RM 路径加载 best_model 文件 |
| 包初始化文件名 | packages/utils/data/___init__.py | 三个下划线；命名空间包可能允许导入，但常规名称为 __init__.py |
| GPU 选择 | init.py 第 23 行；main.py 第 44–46 行 | 当前代码强制使用 CUDA；命令显式指定 -G 0 |
| 数据与模型路径 | preprocess_datasets.py 的 set_paths()；两份 YAML | 使用者仍需修改绝对路径；预处理数据中也包含完整图像路径 |

## 文档与发布信息

- [x] 按作者说明更正为基于 PRN 进一步开发，并区分论文作者与代码作者。
- [x] 保留现有 MIT 文本，说明它不代表已取得 PRN 继承代码的再分发或重新许可授权。
- [ ] 确认原 PRN 版本、继承代码范围及适用许可，补齐最终许可证范围。
- [x] requirements.txt 按作者提供的七个包版本固定。
- [x] README 添加百度网盘权重下载表，已填写作者提供的 model 文件夹分享链接，提取码 rj4z。
- [x] 模型类声明、super 初始化与两份 YAML 已统一为 SGRNet。
- [ ] 稿件 Code availability 仍为占位内容，创建实际仓库后填写准确链接和说明。
- [ ] 对照实际论文实验，说明训练二值图、评估高斯图的阶段差异；本次不改变算法实现。
- [x] 填写作者提供的百度网盘链接和提取码。
- [ ] 验证网盘文件名、下载后的权重与模型兼容性；本次未能访问分享页，未下载核验。
- [ ] 填写仓库地址、论文链接/DOI/发表信息。
- [ ] 确认权重与当前模型可互相加载，并在实际环境运行评估。
- [ ] 上传时排除数据、权重和 __pycache__，保留 requirements.txt。
- [ ] 若需复现消融表，补充对应配置和命令。

论文采用最终轮次权重：MS 400 轮、NCAA 200 轮，不要求另行增加按验证集选择最佳轮次的逻辑。

## 验证范围

此前 17 个 Python 文件通过语法解析；最小函数验证确认标量 mAP 已修复，也确认当前中心公式、训练/评估空间图差异和缺失权重异常。此次文档更新再次核对类名位置，并核对 Python/YAML 文件散列，确认未改动源码和配置。

审查环境没有安装 PyTorch/torchvision，未加载约 800 MB 的权重，未运行完整训练和测试。README 中的论文结果来自作者稿件。
