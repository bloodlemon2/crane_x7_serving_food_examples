# crane_x7_ serving_food_examples
千葉工業大学未来ロボティクス学科の設計製作論3で作成したリポジトリです.  
このパッケージは, ロボットアーム(CRANE-X7)単独での皿のピッキングとRGBDセンサ(RealSenseD435)を併用した色付き皿のピッキングを行うコードをまとめたROS 2パッケージです.
## このパッケージを使用する前に
### 使用環境
- Ubuntu 22.04 LTS
- ROS 2 Humble
### ROS 2及びCRANE-X7のセットアップ  
- ROS 2インストール  
ROS 2は各自でインストールしてください.  
- CRANE-X7及び関連パッケージのインストール  
[RT社公式リポジトリ](https://github.com/rt-net/crane_x7_ros/tree/ros2)よりインストールできます. 以下の手順でインストールしてください.
```
# ROSの環境をセットアップ
$ source /opt/ros/humble/setup.bash

# crane_x7のリポジトリをダウンロード
# ros2_wsは各自のROS 2のワークスペース名に変更してください
$ mkdir -p ~/ros2_ws/src
$ cd ~/ros2_ws/src
$ git clone -b ros2 https://github.com/rt-net/crane_x7_ros.git
$ git clone -b ros2 https://github.com/rt-net/crane_x7_description.git

# 依存パッケージのインストール
$ rosdep install -r -y -i --from-paths .

# ビルドをしてインストール完了
$ cd ~/ros2_ws
$ colcon build --symlink-install
$ source ~/ros2_ws/install/setup.bash
```
([CRANE-X7のREADME](https://github.com/rt-net/crane_x7_ros/tree/ros2/README.md)から転載)  
インストールが完了したら, RT社のパッケージに含まれるサンプルコードをシミュレータ(Gazebo)で試すことができます. シミュレータ使いたい場合は[こちら](https://github.com/rt-net/crane_x7_ros/tree/ros2/crane_x7_examples)を確認してください. 
-  USBポートの設定（実機のCRANE-X7を動かす際に必要となります）
```
# 一時的な付与の場合
$ sudo chmod 666 /dev/ttyUSB0

# 永続的な付与の場合(再起動を伴います)
$ sudo usermod -aG dialout $USER
$ reboot
```
crane_x7_controlの[README](https://github.com/rt-net/crane_x7_ros/blob/ros2/crane_x7_control/README.md)に詳しく書いてあります. 

### RealSenseのセットアップ
[IntelRealSenseのgithub](https://github.com/IntelRealSense/librealsense/blob/development/doc/distribution_linux.md#installing-the-packages)を参照してください. 以下[先ほどのページ](https://github.com/IntelRealSense/librealsense/blob/development/doc/distribution_linux.md#installing-the-packages)から転載
```
# Register the server's public key:
$ sudo mkdir -p /etc/apt/keyrings
$ curl -sSf https://librealsense.intel.com/Debian/librealsense.pgp | sudo tee /etc/apt/keyrings/librealsense.pgp > /dev/null

# Make sure apt HTTPS support is installed:
$ sudo apt-get install apt-transport-https

# Add the server to the list of repositories:
$ echo "deb [signed-by=/etc/apt/keyrings/librealsense.pgp] https://librealsense.intel.com/Debian/apt-repo `lsb_release -cs` main" | \
$ sudo tee /etc/apt/sources.list.d/librealsense.list
$ sudo apt-get update

# Install the libraries (see section below if upgrading packages):
$ sudo apt-get install librealsense2-dkms
$ sudo apt-get install librealsense2-utils

# セットアップできているか確認 (RealSenseを接続していると映像を確認できます):  
$ realsense-viewer
```

# このパッケージの使用方法
## 使用できるサンプル
- plate_pick_and_move
- camera_blue_plate_picking
- camera_yellow_plate_picking
## 用意が必要な物
- お皿(口径140mm)
- 色紙(青色, 黄色)
- のりまたはテープ

お皿は100円ショップなどにあるペーパーボウルを使うと良いです.  
色紙をお皿に貼り付けて色付き皿を作成することで, 色認識でお皿を配膳できます.  
お皿のサイズを変える場合には[これらのコード](https://github.com/bloodlemon2/crane_x7_serving_food_examples/tree/main/src)に変更を加える必要があります.
## インストール方法
```
# このリポジトリをクローン
# ros2_wsは各自のROS 2のワークスペース名に変更してください
$ cd ~/ros2_ws/src
$ git clone https://github.com/bloodlemon2/crane_x7_serving_food_examples.git

# ビルドしてインストール完了
$ cd ~/ros2_ws
$ colcon build
$ source ~/ros2_ws/install/setup.bash


# 'source ~/ros2_ws/install/setup.bash'は, ~/.bashrcに書いておくことを推奨します.   
# 下のコマンドで~/.bashrcに追記できます.  
$ echo 'source ~/ros2_ws/install/setup.bash' >> ~/.bashrc
# ~/.bashrcに'source ~/ros2_ws/install/setup.bash'が書かれていると, 下のコマンドが代わりに使用できます.
$ source ~/.bashrc
```
## 実行手順
シミュレータ(Gazebo)を使用する場合は[こちら](https://github.com/rt-net/crane_x7_ros/blob/ros2/crane_x7_examples/README.md#1-move_group%E3%81%A8gazebo%E3%82%92%E8%B5%B7%E5%8B%95%E3%81%99%E3%82%8B), 実機(CRANE-X7)を使用する場合は[こちら](https://github.com/rt-net/crane_x7_ros/blob/ros2/crane_x7_examples/README.md#3-move_group%E3%81%A8controller%E3%82%92%E8%B5%B7%E5%8B%95%E3%81%99%E3%82%8B)を確認してください.

## plate_pick_and_move
特定の場所にあるお皿を掴み, 配膳するコードです.  
Gazeboで実行する場合, [crane_x7_ros/crane_x7_gazebo/worlds/table.sdf](https://github.com/rt-net/crane_x7_ros/blob/ros2/crane_x7_gazebo/worlds/table.sdf)から木のブロックを削除して, お皿のモデルを追加する必要があります. table.sdfの内容を[こちら](https://github.com/bloodlemon2/crane_x7_serving_food_examples/blob/gazebo/worlds/table.sdf)に変更してから実行してください.  
また, [plate_pick_and_move.cpp](https://github.com/bloodlemon2/crane_x7_serving_food_examples/blob/main/src/plate_pick_and_move.cpp)の103行目をz = 0.09に変更してください.  
次のコマンドで実行できます.
- Gazeboで実行する場合
```
ros2 launch crane_x7_serving_food_examples plate_pick_and_move.launch.py use_sim_time:='true'
```

- 実機で実行する場合
```
ros2 launch crane_x7_serving_food_examples plate_pick_and_move.launch.py
```
#### plate_pick_and_moveのデモ動画
https://github.com/user-attachments/assets/02dbadf3-9645-4934-8fe8-8588b7e20cdd

## camera_blue_plate_picking
RGBDセンサを用いて青色のお皿を掴み, 配膳するコードです.  
RealSenseを接続して実機で実行します. [こちら](https://github.com/rt-net/crane_x7_ros/blob/ros2/crane_x7_examples/README.md#realsense-d435%E3%83%9E%E3%82%A6%E3%83%B3%E3%82%BF%E6%90%AD%E8%BC%89%E3%83%A2%E3%83%87%E3%83%AB%E3%82%92%E4%BD%BF%E7%94%A8%E3%81%99%E3%82%8B%E5%A0%B4%E5%90%88)の'RealSense D435マウンタ搭載モデルを使用する場合'からRVizを起動してから実行してください.  
次のコマンドで実行できます.
```
ros2 launch crane_x7_serving_food_examples camera_blue_plate_picking.launch.py
```
#### camera_blue_plate_pickingのデモ動画
https://github.com/user-attachments/assets/53df314e-9f4c-49b6-84e4-77aebbd0b484

## camera_yellow_plate_picking
RGBDセンサを用いて黄色のお皿を掴み, 配膳するコードです.  
RealSenseを接続して実機で実行します. [こちら](https://github.com/rt-net/crane_x7_ros/blob/ros2/crane_x7_examples/README.md#realsense-d435%E3%83%9E%E3%82%A6%E3%83%B3%E3%82%BF%E6%90%AD%E8%BC%89%E3%83%A2%E3%83%87%E3%83%AB%E3%82%92%E4%BD%BF%E7%94%A8%E3%81%99%E3%82%8B%E5%A0%B4%E5%90%88)のRealSense D435マウンタ搭載モデルを使用する場合でRVizを起動してから実行してください.  
次のコマンドで実行できます.
```
ros2 launch crane_x7_serving_food_examples camera_yellow_plate_picking.launch.py
```
#### camera_yellow_plate_pickingのデモ動画
https://github.com/user-attachments/assets/8e64dbc9-8338-427f-81ba-7d680a01fc7c
# 引継ぎ事項
## 発生する可能性のあるエラーと解決法
- RVizに二値化の画像がでない
    - ROS 2インストール時にOpenCVがインストールされています. 自分で新たにOpenCVをインストールすると異なるバージョンが競合して, RVizに二値化の画像がでなくなることがあります.
    - OpenCVが異なるバージョンが競合していると, ビルドするときにも警告がでます.
    - 自分でインストールしたバージョンのOpenCVアンインストールすることをおすすめします.
## やり残したこと
- GitHub Actionsでのテスト
- GazeboにCADで作成したのモデルの追加
- Gazebo上でRealSenseの動作
- [Gazeboで実行する場合](https://github.com/bloodlemon2/crane_x7_serving_food_examples?tab=readme-ov-file#plate_pick_and_move)でお皿を追加してGazeboを起動するとエラーがでる.
    - '[ign gazebo -r-1] [Err] [SystemPaths.cc:473] Could not resolve file [texture.png]'というエラー内容.
    - シミュレーションをするときに, アームと皿のあたり判定などの問題は無いようでした.
- 一つのプログラムで複数の色を認識し, 色ごとに動作を変える.
# ライセンス
- このパッケージはRT Corporationの公開する[パッケージ](https://github.com/rt-net/crane_x7_ros/tree/ros2)の以下の5つファイルを改変して作成されています.
    - [camera_example.launch.py](https://github.com/rt-net/crane_x7_ros/blob/ros2/crane_x7_examples/launch/camera_example.launch.py)
    - [example.launch.py](https://github.com/rt-net/crane_x7_ros/blob/ros2/crane_x7_examples/launch/example.launch.py)
    - [color_detection.cpp](https://github.com/rt-net/crane_x7_ros/blob/ros2/crane_x7_examples/src/color_detection.cpp)
    - [pick_and_place.cpp](https://github.com/rt-net/crane_x7_ros/blob/ros2/crane_x7_examples/src/pick_and_place.cpp)
    - [pick_and_place_tf.cpp](https://github.com/rt-net/crane_x7_ros/blob/ros2/crane_x7_examples/src/pick_and_place_tf.cpp)
- このソフトウェアパッケージは, Apache License, Version 2.0に基づき公開されています.
- ライセンス全文は[LICENSE](https://github.com/bloodlemon2/crane_x7_ServingFood_examples/blob/main/LICENSE)から確認できます.
- © 2024 Tomoya Tsuji, Kenta Hirachi
