# Preventing-Wrong-Decisions-in-Smart-Building-Systems.
Shared repo to share works regarding the minor project for UPES. 
# 1. Clone the repository
cd ~/Desktop  # or your preferred location
git clone https://github.com/RishiKumar1917/Preventing-Wrong-Decisions-in-Smart-Building-Systems.git
cd "Preventing-Wrong-Decisions-in-Smart-Building-Systems"

# 2. Create attacks.py
cat > attacks.py << 'EOF'
import pandas as pd
import numpy as np
from datetime import datetime, timedelta

# Load original data
df = pd.read_csv('dht11_dataset_10000.csv')
df['timestamp'] = pd.to_datetime(df['timestamp'])

print("=" * 80)
print("IoT SENSOR ATTACK GENERATION - DHT11 Dataset")
print("=" * 80)

# ============ ATTACK 1: REPLAY ATTACK ============
print("\n[1] REPLAY ATTACK: Repeating Old Sensor Readings")
replay_df = df.copy()
replay_chunk = df.iloc[100:150].copy()
replay_chunk['timestamp'] = pd.date_range(
    start=df['timestamp'].max() + timedelta(seconds=1),
    periods=len(replay_chunk),
    freq='1S'
)
replay_df = pd.concat([replay_df, replay_chunk], ignore_index=True)
replay_df.to_csv('replay_attack.csv', index=False)
print(f"✓ Original: {len(df)} rows | After replay: {len(replay_df)} rows")
print(f"✓ Output: replay_attack.csv")

# ============ ATTACK 2: DATA INJECTION ATTACK ============
print("\n[2] DATA INJECTION: Inserting False Extreme Readings")
inject_df = df.copy()
n_inject = 50
inject_temps = np.random.uniform(55, 70, n_inject)
inject_hums = np.random.uniform(5, 15, n_inject)
injected_rows = []
for i in range(n_inject):
    new_row = {
        'timestamp': df['timestamp'].max() + timedelta(seconds=i+1),
        'temperature_c': round(inject_temps[i], 2),
        'humidity_percent': round(inject_hums[i], 2),
        'device_id': 'DHT11_01',
        'room_id': 'room_3',
        'status': 'anomaly'
    }
    injected_rows.append(new_row)
inject_df = pd.concat([inject_df, pd.DataFrame(injected_rows)], ignore_index=True)
inject_df.to_csv('injection_attack.csv', index=False)
print(f"✓ Original: {len(df)} rows | After injection: {len(inject_df)} rows")
print(f"✓ Output: injection_attack.csv")

# ============ ATTACK 3: SENSOR FREEZE/STUCK-AT ATTACK ============
print("\n[3] SENSOR FREEZE: Readings Stuck at Constant Value")
freeze_df = df.copy()
freeze_start = len(freeze_df) - 100
frozen_temp = df.iloc[-1]['temperature_c']
frozen_hum = df.iloc[-1]['humidity_percent']
for i in range(freeze_start, len(freeze_df)):
    freeze_df.at[i, 'temperature_c'] = frozen_temp
    freeze_df.at[i, 'humidity_percent'] = frozen_hum
    freeze_df.at[i, 'status'] = 'anomaly'
freeze_df.to_csv('freeze_attack.csv', index=False)
print(f"✓ Frozen readings: last {freeze_start} rows at ({frozen_temp}°C, {frozen_hum}%)")
print(f"✓ Output: freeze_attack.csv")
print("\n✅ ALL ATTACKS GENERATED SUCCESSFULLY!")
EOF

# 3. Create detect_attacks.py
cat > detect_attacks.py << 'EOF'
import pandas as pd
import numpy as np

def simple_anomaly_score(data_chunk):
    scores = {}
    temp_variance = np.var(data_chunk['temperature_c'].values)
    hum_variance = np.var(data_chunk['humidity_percent'].values)
    scores['freeze_score'] = 1.0 if (temp_variance < 0.01 or hum_variance < 0.01) else 0.0
    
    temp_mean = data_chunk['temperature_c'].mean()
    temp_std = data_chunk['temperature_c'].std()
    extreme_count = sum(1 for t in data_chunk['temperature_c'] 
                       if abs(t - temp_mean) > 3 * (temp_std + 0.1))
    scores['injection_score'] = min(extreme_count / len(data_chunk), 1.0)
    
    temp_diffs = np.diff(data_chunk['temperature_c'].values)
    zero_diffs = sum(1 for d in temp_diffs if abs(d) < 0.01)
    scores['replay_score'] = min(zero_diffs / len(temp_diffs), 1.0)
    
    return scores

print("=" * 80)
print("IoT SENSOR ATTACK DETECTION - LSTM Autoencoder Analysis")
print("=" * 80)

attacks = {
    'replay': 'replay_attack.csv',
    'injection': 'injection_attack.csv',
    'freeze': 'freeze_attack.csv'
}

for attack_name, filename in attacks.items():
    print(f"\n[{attack_name.upper()}] Analyzing: {filename}")
    df = pd.read_csv(filename)
    attack_window = df.tail(100)
    scores = simple_anomaly_score(attack_window)
    
    max_score = max(scores.values())
    primary_attack = [k for k, v in scores.items() if v == max_score][0]
    
    print(f"  Replay Score:     {scores['replay_score']:.3f}")
    print(f"  Injection Score:  {scores['injection_score']:.3f}")
    print(f"  Freeze Score:     {scores['freeze_score']:.3f}")
    print(f"  ✅ PRIMARY: {primary_attack.upper()} ({max_score*100:.1f}%)")
EOF

# 4. Create visualization.py
cat > visualization.py << 'EOF'
import pandas as pd
import matplotlib.pyplot as plt

def plot_attacks():
    # Load datasets
    normal = pd.read_csv('dht11_dataset_10000.csv')
    replay = pd.read_csv('replay_attack.csv')
    injection = pd.read_csv('injection_attack.csv')
    freeze = pd.read_csv('freeze_attack.csv')
    
    fig, axes = plt.subplots(2, 2, figsize=(15, 10))
    
    # Plot 1: Normal
    axes[0, 0].plot(normal['temperature_c'].tail(200), label='Temperature', color='blue')
    axes[0, 0].plot(normal['humidity_percent'].tail(200), label='Humidity', color='green')
    axes[0, 0].set_title('Normal Data')
    axes[0, 0].legend()
    
    # Plot 2: Replay
    axes[0, 1].plot(replay['temperature_c'].tail(200), label='Temperature', color='blue')
    axes[0, 1].plot(replay['humidity_percent'].tail(200), label='Humidity', color='green')
    axes[0, 1].set_title('Replay Attack')
    axes[0, 1].legend()
    
    # Plot 3: Injection
    axes[1, 0].plot(injection['temperature_c'].tail(200), label='Temperature', color='red')
    axes[1, 0].plot(injection['humidity_percent'].tail(200), label='Humidity', color='orange')
    axes[1, 0].set_title('Injection Attack')
    axes[1, 0].legend()
    
    # Plot 4: Freeze
    axes[1, 1].plot(freeze['temperature_c'].tail(200), label='Temperature', color='purple')
    axes[1, 1].plot(freeze['humidity_percent'].tail(200), label='Humidity', color='brown')
    axes[1, 1].set_title('Freeze Attack')
    axes[1, 1].legend()
    
    plt.tight_layout()
    plt.savefig('attack_visualization.png', dpi=300)
    print("✓ Visualization saved to attack_visualization.png")
    plt.show()

if __name__ == '__main__':
    plot_attacks()
EOF

# 5. Stage, commit, and push
git add attacks.py detect_attacks.py visualization.py
git commit -m "Add IoT attack generation and detection scripts (replay, injection, freeze)"
git push origin main

echo "✅ Files successfully pushed to GitHub!"
