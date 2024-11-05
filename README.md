# VIBE: 人体の姿勢と形状推定のためのビデオ推論 [CVPR-2020]
[![report](https://img.shields.io/badge/arxiv-report-red)](https://arxiv.org/abs/1912.05656) [![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/drive/1dFfwxZ52MN86FA6uFNypMEdFShd2euQA) [![PWC](https://img.shields.io/endpoint.svg?url=https://paperswithcode.com/badge/vibe-video-inference-for-human-body-pose-and/3d-human-pose-estimation-on-3dpw)](https://paperswithcode.com/sota/3d-human-pose-estimation-on-3dpw?p=vibe-video-inference-for-human-body-pose-and)

<p float="center">
  <img src="doc/assets/header_1.gif" width="49%" />
  <img src="doc/assets/header_2.gif" width="49%" />
</p>

詳細については、以下のYouTubeビデオをご覧ください。

| 論文解説ビデオ | 定性的結果 |
|------------------------------------------------------------------------------------------------------------|--------------------------------------------------------------------------------------------------------------------|
| [![PaperVideo](https://img.youtube.com/vi/rIr-nX63dUA/0.jpg)](https://www.youtube.com/watch?v=rIr-nX63dUA) | [![QualitativeResults](https://img.youtube.com/vi/fW0sIZfQcIs/0.jpg)](https://www.youtube.com/watch?v=fW0sIZfQcIs) |

> [**VIBE: ビデオからの人体姿勢・形状推定のための推論**](https://arxiv.org/abs/1912.05656),            
> [Muhammed Kocabas](https://ps.is.tuebingen.mpg.de/person/mkocabas), [Nikos Athanasiou](https://ps.is.tuebingen.mpg.de/person/nathanasiou), 
[Michael J. Black](https://ps.is.tuebingen.mpg.de/person/black),        
> *IEEE Computer Vision and Pattern Recognition, 2020*

## 特徴

_**V**ideo **I**nference for **B**ody Pose and Shape **E**stimation_ (VIBE)は、ビデオベースの姿勢・形状推定手法です。
入力ビデオの各フレームに対してSMPL体型モデルのパラメータを予測します。詳細については[arXivレポート](https://arxiv.org/abs/1912.05656)をご参照ください。

この実装の特徴：

- デモと学習コードがPyTorchで純粋に実装されています
- 複数人が写る任意のビデオに対応可能です
- CPUとGPU両方での推論をサポートしています（GPUの方が大幅に高速です）
- RTX2080Tiで最大30FPSの高速処理が可能です（[この表](doc/demo.md#runtime-performance)を参照）
- 3DPWとMPI-INF-3DHPデータセットでSOTAの結果を達成しています
- Temporal SMPLifyの実装を含みます
- 学習コードと一からの学習方法の詳細な手順を含みます
- 主要な3DCGソフトウェアで使用可能なFBX/glTF出力を生成できます

<p float="center">
  <img src="doc/assets/method_1.gif" width="49%" />
  <img src="doc/assets/parkour.gif" width="49%" />
</p>

## 更新情報

- 2021/05/01: [@carlosedubarreto](https://github.com/carlosedubarreto)様のご協力により、Windows用インストールチュートリアルが追加されました
- 2020/10/06: OneEuroFilterによるスムージングをサポート
- 2020/09/14: FBX/glTF変換スクリプトをリリース

## 始め方
VIBEはUbuntu 18.04、Python 3.7以上で実装とテストが行われています。GPUとCPU両方での推論をサポートしています。
適切な環境がない場合は、Colabデモの実行をお試しください。

リポジトリのクローン：
```bash
git clone https://github.com/mkocabas/VIBE.git
```

`virtualenv`または`conda`を使用して必要なパッケージをインストール：
```bash
# pip
source scripts/install_pip.sh

# conda
source scripts/install_conda.sh
```

## デモの実行

任意のビデオでVIBEを実行できる便利なデモコードを用意しています。
まず、必要なデータ（学習済みモデルとSMPLモデルパラメータ）をダウンロードする必要があります：

```bash
source scripts/prepare_data.sh
```

その後、以下のように簡単にデモを実行できます：

```bash
# ローカルビデオで実行
python demo.py --vid_file sample_video.mp4 --output_folder output/ --display

# YouTubeビデオで実行
python demo.py --vid_file https://www.youtube.com/watch?v=wPZP8Bwxplo --output_folder output/ --display
```

デモコードの詳細については[`doc/demo.md`](doc/demo.md)を参照してください。

`--sideview`フラグを使用したデモ出力のサンプル：

<p float="left">
  <img src="doc/assets/sample_video.gif" width="30%" />
</p>

### FBX・glTF出力（新機能！）
VIBEの出力をBlender、Unityなどの3DCGツールで使用可能なスタンドアローンのFBX/glTFファイルに変換するスクリプトを提供しています。
変換スクリプトを実行するには以下の手順に従ってください：

- SMPLボディモデルのFBXファイルをダウンロードする必要があります
    - [SMPLウェブサイト](https://smpl.is.tue.mpg.de/)でアカウントを作成
    - [リンク](https://psfiles.is.tuebingen.mpg.de/downloads/smpl/SMPL_unity_v-1-0-0-zip)からUnity互換のFBXファイルをダウンロード
    - 解凍したファイルを`data/SMPL_unity_v.1.0.0`に配置
- BlenderのPython APIをインストール
    - このスクリプトはBlender v2.8.0およびv2.8.3でテスト済み
- 以下のコマンドでVIBE出力をFBXに変換：
```
python lib/utils/fbx_output.py \
    --input output/sample_video/vibe_output.pkl \
    --output output/sample_video/fbx_output.fbx \ # glTFの場合は拡張子を*.glbに指定
    --fps_source 30 \
    --fps_target 30 \
    --gender <male or female> \
    --person_id <VIBE出力のトラックレットID>
```

### Windowsインストールチュートリアル

[@carlosedubarreto](https://github.com/carlosedubarreto)様提供のWindowsマシンでのVIBEのインストールと実行手順：

- VIBEのWindowsインストールチュートリアル: https://youtu.be/3qhs5IRJ1LI
- FBX変換: https://youtu.be/w1biKeiQThY
- ヘルパーリポジトリ: https://github.com/carlosedubarreto/vibe_win_install

## Google Colab
このプロジェクトを実行するための適切な環境がない場合は、Google Colabをお試しください。
無料でクラウド上でプロジェクトを実行できます。用意したノートブックを使用してColabデモを試すことができます：
[![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/drive/1dFfwxZ52MN86FA6uFNypMEdFShd2euQA)

## 学習
以下のコマンドで学習を開始します：

```shell script
source scripts/prepare_training_data.sh
python train.py --cfg configs/config.yaml
```

データ処理スクリプトを実行する前に、学習用データセットをダウンロードして準備する必要があることに注意してください。
準備方法の詳細については[`doc/train.md`](doc/train.md)を参照してください。

## 評価

ここでは、VIBEと最近の最先端手法を3D姿勢推定データセットで比較します。評価指標は
Procrustes Aligned Mean Per Joint Position Error (PA-MPJPE)で、単位はmmです。

| モデル | 3DPW &#8595; | MPI-INF-3DHP &#8595; | H36M &#8595; |
|----------------|:----:|:------------:|:----:|
| SPIN | 59.2 | 67.5 | **41.1** |
| Temporal HMR | 76.7 | 89.8 | 56.8 |
| VIBE | 56.5 | **63.4** | 41.5 |

この表の結果を再現するか、事前学習済みモデルを評価するには[`doc/eval.md`](doc/eval.md)を参照してください。

**訂正**：データセットの前処理のミスにより、論文の表1における3DPWで学習したVIBEの結果は正しくありませんでした。
また、3DPWでの学習は定量的な性能は向上しますが、定性的な結果は良好ではありません。
arXivバージョンは修正された結果で更新される予定です。

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
このコードは[LICENSEファイル](LICENSE)で定義される**非商用の科学研究目的**でのみ利用可能です。このコードをダウンロードして使用することで、[LICENSE](LICENSE)の条項に同意したものとみなされます。サードパーティのデータセットとソフトウェアはそれぞれのライセンスに従います。

## 参考文献
各ファイル内で外部から借用した関数やスクリプトを明記しています。以下は私たちが参考にした素晴らしいリソースです：

- 事前学習済みHMRと一部の関数は[SPIN](https://github.com/nkolot/SPIN)から借用
- SMPLモデルとレイヤーは[SMPL-Xモデル](https://github.com/vchoutas/smplx)から
- 一部の関数は[Temporal HMR](https://github.com/akanazawa/human_dynamics)から借用
- 一部の関数は[HMR-pytorch](https://github.com/MandyMo/pytorch_HMR)から借用
- 一部の関数は[Kornia](https://github.com/kornia/kornia)から借用
- ポーズトラッカーは[STAF](https://github.com/soulslicer/openpose/tree/staf)から
