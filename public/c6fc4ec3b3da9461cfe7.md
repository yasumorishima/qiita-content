---
title: 大谷翔平選手と他のメジャーリーガーのホームランの打球軌跡アニメーション
tags:
  - pybaseball
  - HiÐΞ
private: false
updated_at: '2023-05-03T11:28:30+09:00'
id: c6fc4ec3b3da9461cfe7
organization_url_name: null
slide: false
ignorePublish: false
---
このコードは、野球選手の大谷翔平選手（ID: 660271）に関するデータを分析し、彼の打撃データを視覚化するためのものです。



***

# statcast関数を使って、2023年の大谷翔平選手の試合データを取得

pybaseballというライブラリをインストールしています。このライブラリを使って、野球の試合データを取得・分析できます。

statcast関数を使って、2023年の大谷翔平選手の試合データを取得しています。

データフレームから、大谷翔平選手の打席データを抽出しています。

<pre><code>!pip install pybaseball
from pybaseball import statcast
df = statcast(start_dt='2023-03-30', end_dt='2023-12-31')
df_ohtani = df[df['batter'] == 660271]
</code></pre>

***
# 'launch_angle'（打球角度）と'launch_speed'（打球速度）を散布図でプロット

抽出したデータから、シングルヒット、ダブルヒット、トリプルヒット、ホームランのデータをそれぞれ取り出しています。
欠損値を取り除いたデータフレームを作成し、打撃データの'launch_angle'（打球角度）と'launch_speed'（打球速度）を散布図でプロットしています。

プロットのタイトル、x軸ラベル、y軸ラベルを設定し、罫線を追加しています。

最後に、作成した散布図を表示しています。

この散布図により、大谷翔平選手の打球角度と打球速度の関係を視覚的に理解することができます。



<pre><code>import matplotlib.pyplot as plt
ohtani_hit_data = df_ohtani[df_ohtani['events'].isin(['single', 'double', 'triple', 'home_run'])]

ohtani_single_data = ohtani_hit_data[ohtani_hit_data['events'] == 'single']
ohtani_double_data = ohtani_hit_data[ohtani_hit_data['events'] == 'double']
ohtani_triple_data = ohtani_hit_data[ohtani_hit_data['events'] == 'triple']
ohtani_home_run_data = ohtani_hit_data[ohtani_hit_data['events'] == 'home_run']

ohtani_hit_data_clean = ohtani_hit_data.dropna(subset=['launch_angle', 'launch_speed'])

plt.scatter(ohtani_hit_data_clean['launch_angle'], ohtani_hit_data_clean['launch_speed'])
plt.xlabel('Launch Angle (degrees)')
plt.ylabel('Launch Speed (mph)')
plt.title('Scatter plot of Launch Angle vs. Launch Speed for Shohei Ohtani')

# 罫線
plt.grid(which='both', linestyle='--', color='gray', alpha=0.5)

plt.show()
</code></pre>

![](https://pbs.twimg.com/media/FvKtVWWaMAEM_kX?format=png&name=small)

***
# 'launch_angle'（打球角度）と'launch_speed'（打球速度）を散布図でプロット2


このコードは、前回のコードで作成した散布図をさらに詳細化し、大谷翔平選手のシングルヒット、ダブルヒット、トリプルヒット、ホームランそれぞれの打球角度と打球速度の関係を視覚的に表示しています。

<pre><code>ohtani_single_data_clean = ohtani_single_data.dropna(subset=['launch_angle', 'launch_speed'])
ohtani_double_data_clean = ohtani_double_data.dropna(subset=['launch_angle', 'launch_speed'])
ohtani_triple_data_clean = ohtani_triple_data.dropna(subset=['launch_angle', 'launch_speed'])
ohtani_home_run_data_clean = ohtani_home_run_data.dropna(subset=['launch_angle', 'launch_speed'])

plt.scatter(ohtani_single_data_clean['launch_angle'], ohtani_single_data_clean['launch_speed'], color='blue', label='Single')
plt.scatter(ohtani_double_data_clean['launch_angle'], ohtani_double_data_clean['launch_speed'], color='green', label='Double')
plt.scatter(ohtani_triple_data_clean['launch_angle'], ohtani_triple_data_clean['launch_speed'], color='orange', label='Triple')
plt.scatter(ohtani_home_run_data_clean['launch_angle'], ohtani_home_run_data_clean['launch_speed'], color='red', label='Home Run')

plt.xlabel('Launch Angle (degrees)')
plt.ylabel('Launch Speed (mph)')
plt.title('Scatter plot of Launch Angle vs. Launch Speed for Shohei Ohtani')
plt.legend()

# 罫線
plt.grid(which='both', linestyle='--', color='gray', alpha=0.5)

plt.show()

</code></pre>

![](https://pbs.twimg.com/media/FvKtYjUaAAMn7ow?format=png&name=small)

***
# シングルヒット、ダブルヒット、トリプルヒット、ホームランそれぞれの打球角度と打球速度の関係

このコードは、前回のコードで分析した大谷翔平選手のデータではなく、2023年のMLB全体の打撃データに基づいて、シングルヒット、ダブルヒット、トリプルヒット、ホームランそれぞれの打球角度と打球速度の関係を視覚的に表示しています。

<pre><code>all_hit_data = df[df['events'].isin(['single', 'double', 'triple', 'home_run'])]

all_single_data = all_hit_data[all_hit_data['events'] == 'single']
all_double_data = all_hit_data[all_hit_data['events'] == 'double']
all_triple_data = all_hit_data[all_hit_data['events'] == 'triple']
all_home_run_data = all_hit_data[all_hit_data['events'] == 'home_run']

all_single_data_clean = all_single_data.dropna(subset=['launch_angle', 'launch_speed'])
all_double_data_clean = all_double_data.dropna(subset=['launch_angle', 'launch_speed'])
all_triple_data_clean = all_triple_data.dropna(subset=['launch_angle', 'launch_speed'])
all_home_run_data_clean = all_home_run_data.dropna(subset=['launch_angle', 'launch_speed'])

plt.scatter(all_single_data_clean['launch_angle'], all_single_data_clean['launch_speed'], color='blue', label='Single', alpha=0.1)
plt.scatter(all_double_data_clean['launch_angle'], all_double_data_clean['launch_speed'], color='green', label='Double', alpha=0.1)
plt.scatter(all_triple_data_clean['launch_angle'], all_triple_data_clean['launch_speed'], color='orange', label='Triple', alpha=0.1)
plt.scatter(all_home_run_data_clean['launch_angle'], all_home_run_data_clean['launch_speed'], color='red', label='Home Run', alpha=0.1)

plt.xlabel('Launch Angle (degrees)')
plt.ylabel('Launch Speed (mph)')
plt.title('Scatter plot of Launch Angle vs. Launch Speed for MLB')
plt.legend()

# 罫線
plt.grid(which='both', linestyle='--', color='gray', alpha=0.5)

plt.show()

</code></pre>

![](https://pbs.twimg.com/media/FvKta-qaYAAAh6y?format=png&name=small)

***

# MLB全体のホームランデータに基づいて、打球距離と打球角度の関係

このコードは、2023年のMLB全体のホームランデータに基づいて、打球距離と打球角度の関係を視覚的に表示しています。

<pre><code>all_home_run_data_clean = all_home_run_data.dropna(subset=['hit_distance_sc', 'launch_angle'])

plt.scatter(all_home_run_data_clean['hit_distance_sc'], all_home_run_data_clean['launch_angle'])
plt.xlabel('Hit Distance (ft)')
plt.ylabel('Launch Angle (degrees)')
plt.title('Scatter plot of Hit Distance vs. Launch Angle for Home Runs')
plt.grid(which='both', linestyle='--', color='gray', alpha=0.5)
plt.show()

</code></pre>

![](https://pbs.twimg.com/media/FvKtc1AaMAIkjYT?format=png&name=small)


***

# MLB全体と大谷翔平選手のホームランデータを比較、打球距離と打球角度の関係

このコードは、2023年のMLB全体のホームランデータと大谷翔平選手のホームランデータを比較し、打球距離と打球角度の関係を視覚的に表示しています。

<pre><code># ft to m conversion
all_home_run_data_clean['hit_distance_m'] = all_home_run_data_clean['hit_distance_sc'] * 0.3048
ohtani_home_run_data_clean['hit_distance_m'] = ohtani_home_run_data_clean['hit_distance_sc'] * 0.3048

plt.scatter(all_home_run_data_clean['hit_distance_m'], all_home_run_data_clean['launch_angle'], label='All Players')
plt.scatter(ohtani_home_run_data_clean['hit_distance_m'], ohtani_home_run_data_clean['launch_angle'], color='red', label='Shohei Ohtani')
plt.xlabel('Hit Distance (m)')
plt.ylabel('Launch Angle (degrees)')
plt.title('Scatter plot of Hit Distance vs. Launch Angle for Home Runs')
plt.grid(which='both', linestyle='--', color='gray', alpha=0.5)
plt.legend()
plt.show()

</code></pre>

![](https://pbs.twimg.com/media/FvKtesTacAEvjup?format=png&name=small)

***

# MLB全体と大谷翔平選手のホームランデータを比較、打球距離と打球速度の関係

このコードは、2023年のMLB全体のホームランデータと大谷翔平選手のホームランデータを比較し、打球距離と打球速度の関係を視覚的に表示しています。

<pre><code>plt.scatter(all_home_run_data_clean['hit_distance_m'], all_home_run_data_clean['launch_speed'], label='All Players')
plt.scatter(ohtani_home_run_data_clean['hit_distance_m'], ohtani_home_run_data_clean['launch_speed'], color='red', label='Shohei Ohtani')
plt.xlabel('Hit Distance (m)')
plt.ylabel('Launch Speed (mph)')
plt.title('Scatter plot of Hit Distance vs. Launch Speed for Home Runs')
plt.grid(which='both', linestyle='--', color='gray', alpha=0.5)
plt.legend()
plt.show()

</code></pre>

![](https://pbs.twimg.com/media/FvKthciaUAAgUXY?format=png&name=small)

***

# 大谷翔平選手のホームランの打球軌跡

このコードは、大谷翔平選手のホームランの打球軌跡を計算し、視覚的に表示しています。

<pre><code>import numpy as np
import matplotlib.pyplot as plt

def compute_trajectory(hit_distance, launch_speed, launch_angle):
    launch_speed = launch_speed * 1609.34 / 3600  # Convert mph to m/s
    launch_angle = np.radians(launch_angle)  # Convert degrees to radians
    g = 9.81  # Gravity acceleration in m/s^2

    t_flight = 2 * launch_speed * np.sin(launch_angle) / g
    t = np.linspace(0, t_flight, num=1000)
    x = launch_speed * np.cos(launch_angle) * t
    y = launch_speed * np.sin(launch_angle) * t - 0.5 * g * t ** 2

    # Scale the x-axis to match the hit_distance
    x = x * (hit_distance / max(x))

    return x, y

# Plot trajectories for each home run
plt.figure(figsize=(10, 6))
for index, row in ohtani_home_run_data_clean.iterrows():
    hit_distance = row['hit_distance_sc'] * 0.3048  # Convert ft to m
    launch_speed = row['launch_speed']
    launch_angle = row['launch_angle']

    x, y = compute_trajectory(hit_distance, launch_speed, launch_angle)
    plt.plot(x, y)

plt.xlabel('Distance (m)')
plt.ylabel('Height (m)')
plt.title('Projectile motion of Shohei Ohtani home runs')
plt.grid(which='both', linestyle='--', color='gray', alpha=0.5)
plt.show()

</code></pre>

![](https://pbs.twimg.com/media/FvKtjeFagAICCE8?format=png&name=small)


***

# MLB全体と大谷翔平選手のホームランの打球軌跡

このコードは、2023年のMLB全体のホームランの打球軌跡と大谷翔平選手のホームランの打球軌跡を視覚的に表示しています。

<pre><code># Plot trajectories for all home runs
plt.figure(figsize=(12, 8))

# Plot all home runs
for index, row in all_home_run_data_clean.iterrows():
    hit_distance = row['hit_distance_sc'] * 0.3048  # Convert ft to m
    launch_speed = row['launch_speed']
    launch_angle = row['launch_angle']

    x, y = compute_trajectory(hit_distance, launch_speed, launch_angle)
    plt.plot(x, y, color='gray', alpha=0.3)

# Plot Ohtani's home runs with a different color
for index, row in ohtani_home_run_data_clean.iterrows():
    hit_distance = row['hit_distance_sc'] * 0.3048  # Convert ft to m
    launch_speed = row['launch_speed']
    launch_angle = row['launch_angle']

    x, y = compute_trajectory(hit_distance, launch_speed, launch_angle)
    plt.plot(x, y, color='red')

plt.xlabel('Distance (m)')
plt.ylabel('Height (m)')
plt.title('Projectile motion of MLB home runs (Ohtani in red)')
plt.grid(which='both', linestyle='--', color='gray', alpha=0.5)
plt.show()

</code></pre>

![](https://pbs.twimg.com/media/FvKtlUUakAIWzbT?format=jpg&name=small)

***

# 大谷翔平選手のホームラン打球軌跡をアニメーション1

このコードは、大谷翔平選手のホームラン打球軌跡をアニメーションとして表示しています。

<pre><code>import numpy as np
import matplotlib.pyplot as plt
from matplotlib.animation import FuncAnimation
from IPython.display import HTML


def compute_trajectory(hit_distance, launch_speed, launch_angle):
    launch_speed = launch_speed * 1609.34 / 3600  # Convert mph to m/s
    launch_angle = np.radians(launch_angle)  # Convert degrees to radians
    g = 9.81  # Gravity acceleration in m/s^2

    t_flight = 2 * launch_speed * np.sin(launch_angle) / g
    t = np.linspace(0, t_flight, num=1000)
    x = launch_speed * np.cos(launch_angle) * t
    y = launch_speed * np.sin(launch_angle) * t - 0.5 * g * t ** 2

    # Scale the x-axis to match the hit_distance
    x = x * (hit_distance / max(x))

    return x, y



def update(frame_number):
    plt.gca().cla()
    plt.gca().set_ylim(0, max_height)
    plt.gca().set_xlim(0, max_distance)
    plt.xlabel('Distance (m)')
    plt.ylabel('Height (m)')
    plt.title('Projectile motion of Shohei Ohtani home runs')
    plt.grid(which='both', linestyle='--', color='gray', alpha=0.5)

    # Create an empty list to store the legend handles
    legend_handles = []

    for index, row in ohtani_home_run_data_clean.iterrows():
        hit_distance = row['hit_distance_sc'] * 0.3048
        launch_speed = row['launch_speed']
        launch_angle = row['launch_angle']

        x, y = compute_trajectory(hit_distance, launch_speed, launch_angle)
        num_points = len(x)

        # Calculate the start and end indices for the trajectory
        start_index = 0
        end_index = min(int(frame_number / len(ohtani_home_run_data_clean) * num_points), num_points)

        # Plot the trajectory segment
        line, = plt.plot(x[start_index:end_index], y[start_index:end_index], lw=2, label=row['game_date'])
        legend_handles.append(line)

    # Add the legend
    plt.legend(handles=legend_handles, loc='upper left', fontsize=8)

    # Plot the last point
    if end_index == num_points:
        plt.scatter(x[-1], y[-1], c='red', marker='o')

max_height = max([compute_trajectory(row['hit_distance_sc'] * 0.3048, row['launch_speed'], row['launch_angle'])[1].max() for _, row in ohtani_home_run_data_clean.iterrows()])
max_distance = ohtani_home_run_data_clean['hit_distance_sc'].max() * 0.3048

fig = plt.figure(figsize=(10, 6))
ani = FuncAnimation(fig, update, frames=len(ohtani_home_run_data_clean) + 1, interval=1000, blit=False)
HTML(ani.to_html5_video())
</code></pre>

<div style="padding:60% 0 0 0;position:relative;"><iframe src="https://player.vimeo.com/video/823210114?h=2ae3f84760&amp;badge=0&amp;autopause=0&amp;player_id=0&amp;app_id=58479" frameborder="0" allow="autoplay; fullscreen; picture-in-picture" allowfullscreen style="position:absolute;top:0;left:0;width:100%;height:100%;" title="ダウンロード"></iframe></div><script src="https://player.vimeo.com/api/player.js"></script>

https://vimeo.com/823210114/2ae3f84760?share=copy




***

# 大谷翔平選手のホームラン打球軌跡をアニメーション2

このコードは、前回と同じ大谷翔平選手のホームラン打球軌跡をアニメーションとして表示していますが、日付がフォーマットされたものとなっています。


<pre><code>import numpy as np
import matplotlib.pyplot as plt
from matplotlib.animation import FuncAnimation
from IPython.display import HTML


def compute_trajectory(hit_distance, launch_speed, launch_angle):
    launch_speed = launch_speed * 1609.34 / 3600  # Convert mph to m/s
    launch_angle = np.radians(launch_angle)  # Convert degrees to radians
    g = 9.81  # Gravity acceleration in m/s^2

    t_flight = 2 * launch_speed * np.sin(launch_angle) / g
    t = np.linspace(0, t_flight, num=1000)
    x = launch_speed * np.cos(launch_angle) * t
    y = launch_speed * np.sin(launch_angle) * t - 0.5 * g * t ** 2

    # Scale the x-axis to match the hit_distance
    x = x * (hit_distance / max(x))

    return x, y



def update(frame_number):
    plt.gca().cla()
    plt.gca().set_ylim(0, max_height)
    plt.gca().set_xlim(0, max_distance)
    plt.xlabel('Distance (m)')
    plt.ylabel('Height (m)')
    plt.title('Projectile motion of Shohei Ohtani home runs')
    plt.grid(which='both', linestyle='--', color='gray', alpha=0.5)

    # Create an empty list to store the legend handles
    legend_handles = []

    for index, row in ohtani_home_run_data_clean.iterrows():
        hit_distance = row['hit_distance_sc'] * 0.3048
        launch_speed = row['launch_speed']
        launch_angle = row['launch_angle']

        x, y = compute_trajectory(hit_distance, launch_speed, launch_angle)
        num_points = len(x)

        # Calculate the start and end indices for the trajectory
        start_index = 0
        end_index = min(int(frame_number / len(ohtani_home_run_data_clean) * num_points), num_points)

        # Plot the trajectory segment
        formatted_date = row['game_date'].strftime('%Y-%m-%d')  # Format the date as 'YYYY-MM-DD'
        line, = plt.plot(x[start_index:end_index], y[start_index:end_index], lw=2, label=formatted_date)
        legend_handles.append(line)

    # Add the legend
    plt.legend(handles=legend_handles, loc='upper left', fontsize=8)

    # Plot the last point
    if end_index == num_points:
        plt.scatter(x[-1], y[-1], c='red', marker='o')

max_height = max([compute_trajectory(row['hit_distance_sc'] * 0.3048, row['launch_speed'], row['launch_angle'])[1].max() for _, row in ohtani_home_run_data_clean.iterrows()])
max_distance = ohtani_home_run_data_clean['hit_distance_sc'].max() * 0.3048

fig = plt.figure(figsize=(10, 6))
ani = FuncAnimation(fig, update, frames=len(ohtani_home_run_data_clean) + 1, interval=1000, blit=False)
HTML(ani.to_html5_video())
</code></pre>


<div style="padding:60% 0 0 0;position:relative;"><iframe src="https://player.vimeo.com/video/823210755?h=9f3938c466&amp;badge=0&amp;autopause=0&amp;player_id=0&amp;app_id=58479" frameborder="0" allow="autoplay; fullscreen; picture-in-picture" allowfullscreen style="position:absolute;top:0;left:0;width:100%;height:100%;" title="ダウンロード (2)"></iframe></div><script src="https://player.vimeo.com/api/player.js"></script>

https://vimeo.com/823210755?share=copy


***

# 大谷翔平選手のホームラン打球軌跡をアニメーション3

このコードは、大谷翔平選手のホームランの打球軌跡のアニメーションを表示します。ただし、打球の軌跡が出現する速さが打球速度によって異なります。打球速度が速いほど、軌跡が早く表示されます。



<pre><code>import numpy as np
import matplotlib.pyplot as plt
from matplotlib.animation import FuncAnimation
from IPython.display import HTML


def compute_trajectory(hit_distance, launch_speed, launch_angle):
    launch_speed = launch_speed * 1609.34 / 3600  # Convert mph to m/s
    launch_angle = np.radians(launch_angle)  # Convert degrees to radians
    g = 9.81  # Gravity acceleration in m/s^2

    t_flight = 2 * launch_speed * np.sin(launch_angle) / g
    t = np.linspace(0, t_flight, num=1000)
    x = launch_speed * np.cos(launch_angle) * t
    y = launch_speed * np.sin(launch_angle) * t - 0.5 * g * t ** 2

    # Scale the x-axis to match the hit_distance
    x = x * (hit_distance / max(x))

    return x, y



def update(frame_number):
    plt.gca().cla()
    plt.gca().set_ylim(0, max_height)
    plt.gca().set_xlim(0, max_distance)
    plt.xlabel('Distance (m)')
    plt.ylabel('Height (m)')
    plt.title('Projectile motion of Shohei Ohtani home runs')
    plt.grid(which='both', linestyle='--', color='gray', alpha=0.5)

    # Create an empty list to store the legend handles
    legend_handles = []

    for index, row in ohtani_home_run_data_clean.iterrows():
        hit_distance = row['hit_distance_sc'] * 0.3048
        launch_speed = row['launch_speed']
        launch_angle = row['launch_angle']

        x, y = compute_trajectory(hit_distance, launch_speed, launch_angle)
        num_points = len(x)

        # Calculate the start and end indices for the trajectory based on the relative launch speed
        relative_speed = launch_speed / max(ohtani_home_run_data_clean['launch_speed'])
        end_index = min(int(frame_number * relative_speed * num_points / len(ohtani_home_run_data_clean)), num_points)

        # Plot the trajectory segment
        formatted_date = row['game_date'].strftime('%Y-%m-%d')  # Format the date as 'YYYY-MM-DD'
        line, = plt.plot(x[:end_index], y[:end_index], lw=2, label=formatted_date)
        legend_handles.append(line)

    # Add the legend
    plt.legend(handles=legend_handles, loc='upper left', fontsize=8)

    # Plot the last point
    if end_index == num_points:
        plt.scatter(x[-1], y[-1], c='red', marker='o')

max_height = max([compute_trajectory(row['hit_distance_sc'] * 0.3048, row['launch_speed'], row['launch_angle'])[1].max() for _, row in ohtani_home_run_data_clean.iterrows()])
max_distance = ohtani_home_run_data_clean['hit_distance_sc'].max() * 0.3048

fig = plt.figure(figsize=(10, 6))
# Increase the number of frames by a factor of 2 to accommodate the slowest home run
ani = FuncAnimation(fig, update, frames=2 * len(ohtani_home_run_data_clean), interval=1000, blit=False)
HTML(ani.to_html5_video())
</code></pre>


<div style="padding:60% 0 0 0;position:relative;"><iframe src="https://player.vimeo.com/video/823210891?h=8c6310ce78&amp;badge=0&amp;autopause=0&amp;player_id=0&amp;app_id=58479" frameborder="0" allow="autoplay; fullscreen; picture-in-picture" allowfullscreen style="position:absolute;top:0;left:0;width:100%;height:100%;" title="ダウンロード (3)"></iframe></div><script src="https://player.vimeo.com/api/player.js"></script>

https://vimeo.com/823210891?share=copy


***

# 大谷翔平選手のホームラン打球軌跡をアニメーション4

このコードは、大谷翔平選手のホームランの打球軌跡のアニメーションを表示します。ただし、このバージョンでは、アニメーションがより詳細になるようにフレーム数を増やしています。具体的には、フレーム数を以前のコードの4倍にしています。また、intervalパラメータを250に設定して、アニメーションの速度を早くしています。


<pre><code>import numpy as np
import matplotlib.pyplot as plt
from matplotlib.animation import FuncAnimation
from IPython.display import HTML


def compute_trajectory(hit_distance, launch_speed, launch_angle):
    launch_speed = launch_speed * 1609.34 / 3600  # Convert mph to m/s
    launch_angle = np.radians(launch_angle)  # Convert degrees to radians
    g = 9.81  # Gravity acceleration in m/s^2

    t_flight = 2 * launch_speed * np.sin(launch_angle) / g
    t = np.linspace(0, t_flight, num=1000)
    x = launch_speed * np.cos(launch_angle) * t
    y = launch_speed * np.sin(launch_angle) * t - 0.5 * g * t ** 2

    # Scale the x-axis to match the hit_distance
    x = x * (hit_distance / max(x))

    return x, y



def update(frame_number):
    plt.gca().cla()
    plt.gca().set_ylim(0, max_height)
    plt.gca().set_xlim(0, max_distance)
    plt.xlabel('Distance (m)')
    plt.ylabel('Height (m)')
    plt.title('Projectile motion of Shohei Ohtani home runs')
    plt.grid(which='both', linestyle='--', color='gray', alpha=0.5)

    # Create an empty list to store the legend handles
    legend_handles = []

    for index, row in ohtani_home_run_data_clean.iterrows():
        hit_distance = row['hit_distance_sc'] * 0.3048
        launch_speed = row['launch_speed']
        launch_angle = row['launch_angle']

        x, y = compute_trajectory(hit_distance, launch_speed, launch_angle)
        num_points = len(x)

        # Calculate the start and end indices for the trajectory based on the relative launch speed
        relative_speed = launch_speed / max(ohtani_home_run_data_clean['launch_speed'])
        end_index = min(int(frame_number * relative_speed * num_points / len(ohtani_home_run_data_clean)), num_points)

        # Plot the trajectory segment
        formatted_date = row['game_date'].strftime('%Y-%m-%d')  # Format the date as 'YYYY-MM-DD'
        line, = plt.plot(x[:end_index], y[:end_index], lw=2, label=formatted_date)
        legend_handles.append(line)

    # Add the legend
    plt.legend(handles=legend_handles, loc='upper left', fontsize=8)

    # Plot the last point
    if end_index == num_points:
        plt.scatter(x[-1], y[-1], c='red', marker='o')

max_height = max([compute_trajectory(row['hit_distance_sc'] * 0.3048, row['launch_speed'], row['launch_angle'])[1].max() for _, row in ohtani_home_run_data_clean.iterrows()])
max_distance = ohtani_home_run_data_clean['hit_distance_sc'].max() * 0.3048

fig = plt.figure(figsize=(10, 6))
# Increase the number of frames by a factor of 4 to make the animation more detailed
ani = FuncAnimation(fig, update, frames=4 * len(ohtani_home_run_data_clean), interval=250, blit=False)
HTML(ani.to_html5_video())
</code></pre>


<div style="padding:60% 0 0 0;position:relative;"><iframe src="https://player.vimeo.com/video/823211177?h=8444965ca1&amp;badge=0&amp;autopause=0&amp;player_id=0&amp;app_id=58479" frameborder="0" allow="autoplay; fullscreen; picture-in-picture" allowfullscreen style="position:absolute;top:0;left:0;width:100%;height:100%;" title="ダウンロード (4)"></iframe></div><script src="https://player.vimeo.com/api/player.js"></script>


https://vimeo.com/823211177?share=copy


***

# 大谷翔平選手と他のメジャーリーガーのホームランの打球軌跡アニメーション



このコードは、大谷翔平選手のホームランと他のメジャーリーガーのホームランの打球軌跡のアニメーションを表示します。大谷翔平選手のホームランの軌跡は赤色で、他のメジャーリーガーのホームランの軌跡は灰色で表示されます。


<pre><code>import numpy as np
import matplotlib.pyplot as plt
from matplotlib.animation import FuncAnimation
from IPython.display import HTML

def compute_trajectory(hit_distance, launch_speed, launch_angle):
    launch_speed = launch_speed * 1609.34 / 3600  # Convert mph to m/s
    launch_angle = np.radians(launch_angle)  # Convert degrees to radians
    g = 9.81  # Gravity acceleration in m/s^2

    t_flight = 2 * launch_speed * np.sin(launch_angle) / g
    t = np.linspace(0, t_flight, num=1000)
    x = launch_speed * np.cos(launch_angle) * t
    y = launch_speed * np.sin(launch_angle) * t - 0.5 * g * t ** 2

    # Scale the x-axis to match the hit_distance
    x = x * (hit_distance / max(x))

    return x, y

def update(frame_number):
    plt.gca().cla()
    plt.gca().set_ylim(0, max_height)
    plt.gca().set_xlim(0, max_distance)
    
    plt.xlabel('Distance (m)')
    plt.ylabel('Height (m)')
    plt.title('Projectile motion of MLB home runs (Ohtani in red)')
    plt.grid(which='both', linestyle='--', color='gray', alpha=0.5)

    for index, row in all_home_run_data_clean.iterrows():
        hit_distance = row['hit_distance_sc'] * 0.3048
        launch_speed = row['launch_speed']
        launch_angle = row['launch_angle']

        x, y = compute_trajectory(hit_distance, launch_speed, launch_angle)
        plt.plot(x[:frame_number], y[:frame_number], color='gray', alpha=0.3)

    for index, row in ohtani_home_run_data_clean.iterrows():
        hit_distance = row['hit_distance_sc'] * 0.3048
        launch_speed = row['launch_speed']
        launch_angle = row['launch_angle']

        x, y = compute_trajectory(hit_distance, launch_speed, launch_angle)
        plt.plot(x[:frame_number], y[:frame_number], color='red')

max_height = max([compute_trajectory(row['hit_distance_sc'] * 0.3048, row['launch_speed'], row['launch_angle'])[1].max() for _, row in all_home_run_data_clean.iterrows()])
max_distance = all_home_run_data_clean['hit_distance_sc'].max() * 0.3048

fig = plt.figure(figsize=(12, 8))
ani = FuncAnimation(fig, update, frames=1000, interval=20, blit=False)
HTML(ani.to_html5_video())

</code></pre>

<div style="padding:66.67% 0 0 0;position:relative;"><iframe src="https://player.vimeo.com/video/823211249?h=11a10d8d38&amp;badge=0&amp;autopause=0&amp;player_id=0&amp;app_id=58479" frameborder="0" allow="autoplay; fullscreen; picture-in-picture" allowfullscreen style="position:absolute;top:0;left:0;width:100%;height:100%;" title="ダウンロード (5)"></iframe></div><script src="https://player.vimeo.com/api/player.js"></script>

https://vimeo.com/823211249?share=copy
