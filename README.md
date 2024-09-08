# StoryNodeGuide

<img width="945" alt="Story logo" src="https://github.com/user-attachments/assets/66263386-bff3-4594-aa8e-e7ee69385a42">

# Step by step for setup Story IP node and validator
ขั้นตอนการรันโหนด Story IP แบบ step by step

-----------------------------------------------------------------

## ความต้องการของระบบ ตามนี้
|-|-
| CPU | 4 Cores |
| Memory | 8 GB |
| Storage | 200 GB |
| Bandwidth | 10 MBit/s |
| OS | Linux |

-----------------------------------------------------------------

## เริ่มติดตั้งโหนด
### 1. เริ่มจากติดตั้งแพคเกจที่จำเป็นก่อน
```bash
sudo apt update && sudo apt upgrade -y
sudo apt install curl git jq build-essential gcc unzip wget lz4 -y
```

### 2. ติดตั้ง Go
```bash
cd $HOME && \
ver="1.22.0" && \
wget "https://golang.org/dl/go$ver.linux-amd64.tar.gz" && \
sudo rm -rf /usr/local/go && \
sudo tar -C /usr/local -xzf "go$ver.linux-amd64.tar.gz" && \
rm "go$ver.linux-amd64.tar.gz" && \
echo "export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin" >> ~/.bash_profile && \
source ~/.bash_profile && \
go version
```

ต่อไปนี้จะเริ่มติดตั้ง Binary ของ Node นะครับ มี 2 Binary คือ Story-Geth และ Story

### 3. ดาวโหลดและติดตั้ง Story-Geth
```bash
wget -q https://story-geth-binaries.s3.us-west-1.amazonaws.com/geth-public/geth-linux-amd64-0.9.2-ea9f0d2.tar.gz -O /tmp/geth-linux-amd64-0.9.2-ea9f0d2.tar.gz
tar -xzf /tmp/geth-linux-amd64-0.9.2-ea9f0d2.tar.gz -C /tmp
[ ! -d "$HOME/go/bin" ] && mkdir -p $HOME/go/bin
sudo cp /tmp/geth-linux-amd64-0.9.2-ea9f0d2/geth $HOME/go/bin/story-geth
```

### 4. ดาวโหลดและติดตั้ง Story
```bash
wget -q https://story-geth-binaries.s3.us-west-1.amazonaws.com/story-public/story-linux-amd64-0.9.11-2a25df1.tar.gz -O /tmp/story-linux-amd64-0.9.11-2a25df1.tar.gz
tar -xzf /tmp/story-linux-amd64-0.9.11-2a25df1.tar.gz -C /tmp
sudo cp /tmp/story-linux-amd64-0.9.11-2a25df1/story $HOME/go/bin/story
```

### 5. เริ่ม อินนิเชียลไลซ์ Story Iliad Network Node
```bash
$HOME/go/bin/story init --network iliad --moniker "your_moniker"
```

### 6. สร้างและคอนฟิกเซอร์วิส เพื่อรัน Story-Geth
```bash
sudo tee /etc/systemd/system/story-geth.service > /dev/null <<EOF
[Unit]
Description=Story Geth Client
After=network.target

[Service]
User=root
ExecStart=$HOME/go/bin/story-geth --iliad --syncmode full
Restart=on-failure
RestartSec=3
LimitNOFILE=65535

[Install]
WantedBy=multi-user.target
EOF
```

### 7. สร้างและคอนฟิกเซอร์วิส เพื่อรัน Story
```bash
sudo tee /etc/systemd/system/story.service > /dev/null <<EOF
[Unit]
Description=Story Consensus Client
After=network.target

[Service]
User=root
ExecStart=$HOME/go/bin/story run
Restart=on-failure
RestartSec=3
LimitNOFILE=65535

[Install]
WantedBy=multi-user.target
EOF
```

### 8. รีโหลด systemd, Enable, และ Start Services ต้องทำทุกครั้งเมื่อมีการอัพเดทหรือแก้ไขคอนฟิกต่าง ๆ
```bash
sudo systemctl daemon-reload
sudo systemctl enable story-geth story
sudo systemctl start story-geth story
```

### 9. ตรวจสอบสถานะของโหนดว่าสถานะ running หรือไม่
ตรวจสอบสถานะ Story-Geth
```bash
sudo systemctl status story-geth --no-pager -l
```
ตรวจสอบสถานะ Story
```bash
sudo systemctl status story --no-pager -l
```

### 10. ดู Logs
ดู Log ของ Story-Geth
```bash
sudo journalctl -u story-geth -f -o cat
```
ดู Log ของ Story
```bash
sudo journalctl -u story -f -o cat
```

### 11. ตรวจสอบว่าโหนดเราซิงค์ถึงบล็อกปัจจุบันแล้วหรือไม่

```bash
curl -s localhost:26657/status | jq
```
ถ้าในบรรทัด `catching_up` เป็น `true` แสดงว่าโหนดเรายังซิงค์ไม่เสร็จหรือซิงค์อยู่ แต่ถ้าขึ้นว่า `false` นั่นคือซิงค์เสร็จแล้ว (ต้องรอให้ซิงค์เสร็จก่อน หรือเป็น `false` ก่อนถึงจะทำขั้นตอนสร้าง Validator ได้)
> [!CAUTION]
> **สำคัญมาก** ต้องให้โหนดเราซิงค์เสร็จก่อนถึงจะทำขั้นตอนต่อไปในการสร้่าง Validator

-----------------------------------------------------------------

## สร้าง Validator (หลังจากซิงค์เสร็จแล้ว)
### 1. ดึง Public Key และ Private Key ออกมาแสดง ให้เราก๊อปปี้เก็บไว้ด้วย โดยปกติเมื่อเราอินนิเชียลไลซ์โหนด่(ขั้นตอนที่ 5) โปรแกรมจะสร้างมาให้อัตโนมัติ
```bash
story validator export --export-evm-key
```
> [!CAUTION]
> *** ก๊อปปี้เก็บไว้ด้วย ทั้ง Public Key และ Private Key

### 2. ไปของเหรียญ IP จาก faucet โดยเราต้อง connect X ก่อน โดยขอได้ 1 IP ต่อ 24 ชั่วโมง
โดยการสร้าง Validator ต้องใช้ขั้นต่ำ 1 IP (1000000000000000000) เพื่อ Stake เริ่มต้นของ Validator ของเรา แต่ว่าในการสร้าง Validator ต้องจ่ายค่าแก๊สด้วย ดังนั้นควรจะมีมากกว่า 1 เช่น 1.1 เป็นต้น

<a href="https://faucet.story.foundation/" target="_blank">
  <img src="https://github.com/user-attachments/assets/b76fa86a-6c14-4c64-b96f-743d1fe4b73e" alt="Story Faucet" width="150" border="10" />
</a>

### 3. เริ่มสร้าง Validator เลย
```bash
story validator create --stake 1000000000000000000 --private-key "your_private_key"
```
> [!CAUTION]
> พอสร้างเสร็จแล้วควรจะแบ๊คอัพไฟล์ Private key ของ Validator เรา คือไฟล์ `priv_validator_key.json` ซึ่งอยู่ที่ `$HOME/.0gchain/config/`. หรือใช้คำสั่ง cat $HOME/.0gchain/config/priv_validator_key.json แล้วก๊อปเก็บเอาไว้ก็ได้

### ดู Hex Address ของ Validator เรา

```bash
curl -s localhost:26657/status | jq -r '.result.validator_info.address'
```
Hex Address ที่ได้นี้ให้เราก๊อปปี้ไปวางใน [explorer](https://testnet.story.explorers.guru/) ที่ช่องค้นหา เราจะเห็น Validator ของเรา โดยมีรายละเอียดต่าง ๆ ให้เห็น

##################################################################################################
<img width="433" alt="Screenshot 2567-09-08 at 09 13 17" src="https://github.com/user-attachments/assets/aee0bcb0-a7f2-481f-bfed-34d1c6d4690e">
##################################################################################################

-----------------------------------------------------------------

# Command ต่าง ๆ ที่จำเป็น
### ดู Public Key ของ Validator เรา
```bash
curl -s localhost:26657/status | jq -r '.result.validator_info.pub_key.value'
```

### Stake Delegate IP ให้กับ Validator เรา
เปลี่ยนค่าส่วนของ `$VALIDATOR_PUB_KEY_IN_BASE64` เป็นค่าของ Public Key ของ Validator เราและ `$PRIVATE_KEY` คือ Private key ของกระเป๋าที่เราจะใช้ Stake
```bash
story validator stake \
   --validator-pubkey "$VALIDATOR_PUB_KEY_IN_BASE64" \
   --stake 1000000000000000000 \
   --private-key $PRIVATE_KEY
```

### ถอนการติดตั้งโหนด หรือยกเลิกการรันโหนด
> [!CAUTION]
> ก่อนจะทำไม่ว่ากรณีใด ๆ ควรจะสำรองข้อมูลไฟล์สำคัญก่อน
```bash
sudo systemctl stop story-geth
sudo systemctl stop story
sudo systemctl disable story-geth
sudo systemctl disable story
sudo rm /etc/systemd/system/story-geth.service
sudo rm /etc/systemd/system/story.service
sudo systemctl daemon-reload
sudo rm -rf $HOME/.story
sudo rm $HOME/go/bin/story-geth
sudo rm $HOME/go/bin/story
```
