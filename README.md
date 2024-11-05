# VIBE: ビデオからの人体姿勢・形状推定 [CVPR-2020]

[![report](https://img.shields.io/badge/arxiv-report-red)](https://arxiv.org/abs/1912.05656) [![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/drive/1dFfwxZ52MN86FA6uFNypMEdFShd2euQA) [![PWC](https://img.shields.io/endpoint.svg?url=https://paperswithcode.com/badge/vibe-video-inference-for-human-body-pose-and/3d-human-pose-estimation-on-3dpw)](https://paperswithcode.com/sota/3d-human-pose-estimation-on-3dpw?p=vibe-video-inference-for-human-body-pose-and)

詳細については、以下のYouTubeビデオをご覧ください。

## 特徴

**V**ideo **I**nference for **B**ody Pose and Shape **E**stimation (VIBE)は、ビデオベースの姿勢・形状推定手法です。入力ビデオの各フレームに対してSMPL体型モデルのパラメータを予測します。詳細は[arXivの論文](https://arxiv.org/abs/1912.05656)をご参照ください。

この実装の特徴:

- デモと学習コードがPyTorchで純粋に実装されています
- 複数人が写る任意のビデオに対応可能です
- CPUとGPU両方での推論をサポートしています(ただしGPUの方が圧倒的に高速です)
- RTX2080Tiで最大30FPSと高速です([この表](doc/demo.md#runtime-performance)参照)
- 3DPWとMPI-INF-3DHPデータセットでSOTA(最高精度)を達成
- Temporal SMPLifyの実装を含みます  
- 一からの学習に必要なトレーニングコードと詳細な手順を含みます
- 主要な3DCGソフトで使用可能なFBX/glTF出力を生成できます

## アップデート

- 2021/05/01: [@carlosedubarreto](https://github.com/carlosedubarreto)氏の協力によりWindows向けインストール手順が追加されました
- 2020/10/06: OneEuroFilterによるスムージングをサポート
- 2020/09/14: FBX/glTF変換スクリプトがリリースされました

## 始め方

VIBEはUbuntu 18.04、Python 3.7以上で実装・テストされています。GPUとCPU両方での推論をサポートしています。
適切な環境がない場合は、Google Colabデモをお試しください。

リポジトリのクローン:
```bash
git clone https://github.com/mkocabas/VIBE.git
```

`virtualenv`または`conda`で必要なパッケージをインストール:
```bash
# pip
source scripts/install_pip.sh

# conda
source scripts/install_conda.sh
```

## デモの実行

任意のビデオでVIBEを実行できるデモコードを用意しています。
まず、必要なデータ(学習済みモデルとSMPLモデルパラメータ)をダウンロードする必要があります:

```bash
source scripts/prepare_data.sh
```

その後、以下のように簡単にデモを実行できます:

```bash
# ローカルのビデオファイルで実行
python demo.py --vid_file sample_video.mp4 --output_folder output/ --display

# YouTubeのビデオで実行
python demo.py --vid_file https://www.youtube.com/watch?v=wPZP8Bwxplo --output_folder output/ --display
```

デモコードの詳細については[`doc/demo.md`](doc/demo.md)を参照してください。

### FBX/glTF出力(新機能!)

VIBEの出力をBlender、Unityなどの3DCGツールで使用できるスタンドアロンのFBX/glTFファイルに変換するスクリプトを提供しています。変換スクリプトを実行するには以下の手順に従ってください:

- SMPLボディモデル用のFBXファイルをダウンロードする必要があります
    - [SMPLウェブサイト](https://smpl.is.tue.mpg.de/)でアカウントを作成
    - [リンク](https://psfiles.is.tuebingen.mpg.de/downloads/smpl/SMPL_unity_v-1-0-0-zip)からUnity互換のFBXファイルをダウンロード
    - 解凍して`data/SMPL_unity_v.1.0.0`に配置
- Blender Python APIをインストール
    - Blender v2.8.0およびv2.8.3でテスト済み
- 以下のコマンドでVIBE出力をFBXに変換:
```bash
python lib/utils/fbx_output.py \
    --input output/sample_video/vibe_output.pkl \
    --output output/sample_video/fbx_output.fbx \ # glTFの場合は拡張子を*.glbに
    --fps_source 30 \
    --fps_target 30 \
    --gender <male or female> \
    --person_id <tracklet id from VIBE output>
```

### Windowsインストール手順

[@carlosedubarreto](https://github.com/carlosedubarreto)氏による、Windows環境でのVIBEのインストールと実行手順が用意されています:

- VIBEのWindowsインストール手順: https://youtu.be/3qhs5IRJ1LI
- FBX変換: https://youtu.be/w1biKeiQThY  
- 補助用リポジトリ: https://github.com/carlosedubarreto/vibe_win_install

## Google Colab

適切な環境がない場合は、Google Colabを試すことができます。
無料でクラウド上でプロジェクトを実行できます。用意したノートブックでColabデモを試すことができます:
[![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/drive/1dFfwxZ52MN86FA6uFNypMEdFShd2euQA)

## トレーニング

以下のコマンドで学習を開始できます:

```bash
source scripts/prepare_training_data.sh
python train.py --cfg configs/config.yaml
```

データ処理スクリプトを実行する前に、学習データセットをダウンロードして準備する必要があります。
準備方法の詳細は[`doc/train.md`](doc/train.md)を参照してください。

## 評価

ここでは、VIBEと最新のSOTA手法との3D姿勢推定データセットでの比較を示します。評価指標は
Procrustes Aligned Mean Per Joint Position Error (PA-MPJPE、単位:mm)です。

| モデル         | 3DPW &#8595; | MPI-INF-3DHP &#8595; | H36M &#8595; |
|----------------|:----:|:------------:|:----:|
| SPIN           | 59.2 |     67.5     | **41.1** |
| Temporal HMR   | 76.7 |     89.8     | 56.8 |
| VIBE           | 56.5 |     **63.4**     | 41.5 |

この表の結果を再現したり、事前学習済みモデルを評価したりする方法については[`doc/eval.md`](doc/eval.md)を参照してください。

**訂正**: データセット前処理のミスにより、論文の表1に示された3DPWでのVIBEの学習結果は正しくありません。
また、3DPWでの学習は定量的な性能は向上しますが、定性的には良い結果が得られません。arXivバージョンは修正された結果で更新される予定です。

## 引用

```bibtex
@inproceedings{kocabas2019vibe,
  title={VIBE: Video Inference for Human Body Pose and Shape Estimation},
  author={Kocabas, Muhammed and Athanasiou, Nikos and Black, Michael J.},
  booktitle = {The IEEE Conference on Computer Vision and Pattern Recognition (CVPR)},
  month = {June},
  year = {2020}
}
```

## ライセンス

このコードは[LICENSEファイル](LICENSE)で定義される**非商用の科学研究目的**でのみ利用可能です。このコードをダウンロードして使用することで、[LICENSE](LICENSE)の条項に同意したものとみなされます。サードパーティのデータセットとソフトウェアは、それぞれのライセンスに従います。

## 参考文献

各ファイル内で外部から借用した関数やスクリプトを明記しています。以下は参考にした優れたリソースです:

- 事前学習済みHMRと一部の関数は[SPIN](https://github.com/nkolot/SPIN)から借用
- SMPLモデルとレイヤーは[SMPL-Xモデル](https://github.com/vchoutas/smplx)から
- 一部の関数は[Temporal HMR](https://github.com/akanazawa/human_dynamics)から借用
- 一部の関数は[HMR-pytorch](https://github.com/MandyMo/pytorch_HMR)から借用
- 一部の関数は[Kornia](https://github.com/kornia/kornia)から借用  
- ポーズトラッカーは[STAF](https://github.com/soulslicer/openpose/tree/staf)から
