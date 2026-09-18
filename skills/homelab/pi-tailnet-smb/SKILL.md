---
name: pi-tailnet-smb
description: "Use when Pi SMB over Tailscale fails or needs setup."
version: 1.0.0
metadata:
  hermes:
    tags: [toya-claw, homelab, samba, tailscale]
---

# Pi Tailnet SMB

Mac などから **Tailscale IP 経由で Pi に SMB** する／「繋がらない」を直すときの手順。  
関連: `raspberry-pi-tailscale-access`（到達性・SSH 正本は Notion）、Host-Network Act。

## いつ使う

- Mac Finder の `smb://100.x.x.x/...` が失敗する
- Pi に Samba を入れる／Tailnet 限定にしたい
- LAN（eth0）には出したくない

## 診断（先に実測）

```bash
hostname; tailscale status
systemctl is-active smbd nmbd tailscaled
ss -lntp | grep -E ':445|:139|:22'
dpkg -l samba 2>/dev/null | tail -1
nc -vz 127.0.0.1 445; nc -vz "$(tailscale ip -4)" 445
```

| 症状 | 意味 |
|------|------|
| `:22` OK・`:445` refused | Tailscale は生きている。**smbd 未導入／停止**が多い |
| `samba` 未インストール | サーバ本体が無い（client lib だけでは不可） |
| Tailscale offline | 先に TS を直す（本 skill の前） |

## 採用構成（実績・2026-07-31）

**要件:** Tailnet 内デバイスのみ。LAN からは拒否。

### なぜ bind-only だけでは足りないか

- `interfaces = tailscale0` + `bind interfaces only = yes` だと **IPv4 が lo だけ**になりがち
- Tailscale は **POINTOPOINT**。`100.x/32` を interfaces に書いても **smbd が IPv4 に bind しない**ことがある
- **Tailscale ACL だけでは LAN を止められない**（ACL は tailnet トラフィック用。`0.0.0.0:445` なら eth0 は別経路）

### レイヤ（これが本命）

1. **Samba** `hosts allow = 127.0.0.1 100.64.0.0/10 ::1 fd7a:115c:a1e0::/48` / guest 不可 / SMB2+
2. **iptables/ip6tables** TCP/445 は `lo` + `tailscale0` 以外 **REJECT**（永続 unit）
3. **nmbd masked**（NetBIOS 不要。`smb ports = 445`）
4. macOS: `vfs objects = fruit streams_xattr` + fruit_*

永続 FW:

- live: `/usr/local/sbin/smb-tailnet-firewall.sh` + `smb-tailnet-firewall.service`
- skill copies: `scripts/smb-tailnet-firewall.sh` / `templates/smb.conf`

### 共有（現行）

| 名 | path | 備考 |
|----|------|------|
| `home` | `/home/tcaret2` | `veto files = /.ssh/.gnupg/`。`.hermes` 等は見えるので常用は Documents 推奨 |
| `Documents` | `/home/tcaret2/Documents` | 狭い共有 |

ユーザー: `tcaret2`（`smbpasswd`）。Linux パスワードと別。

### パスワード

- **Discord に貼らない**
- 初回: `~/.hermes/samba-smb-password`（mode 600）を Pi 上で `cat`
- 変更: `sudo smbpasswd tcaret2`

### Mac 接続

```text
smb://tcaret2@100.86.189.44/home
smb://tcaret2@100.86.189.44/Documents
# MagicDNS が解けない環境あり → Tailscale IP 優先（Host-Network Act と同じ）
```

### 検証

```bash
PASS=$(cat ~/.hermes/samba-smb-password)
smbclient -L //$(tailscale ip -4) -U tcaret2%"$PASS" -m SMB3
smbclient //$(tailscale ip -4)/Documents -U tcaret2%"$PASS" -m SMB3 -c 'ls'
sudo iptables -L INPUT -n -v | grep smb-tailnet
# 自ホスト→eth0 IP の nc 成功は lo ヘアピン。LAN 拒否は docker/netns 等で測る
```

## Tailscale「だけ」で足りるか

| 手段 | 結論 |
|------|------|
| ACL | tailnet **内**の誰が 445 に届くか。LAN 遮断にはならない |
| `tailscale serve --tcp 445` + smbd=`127.0.0.1` | Tailscale 寄りの代替。Samba 自体は必要 |
| Funnel / Taildrop | SMB マウント代替ではない |

「つながっているから iptables 方式でよい」なら **勝手に Serve 化しない**。明示依頼時のみ。

## ピットフォール

1. **bind interfaces only + tailscale0 で IPv4 が落ちる** → FW + hosts allow
2. **自機→eth0 IP の nc 成功 = LAN 開放と誤読** → ヘアピン
3. **パスワードをチャットに出す** → ローカル 600 ファイル
4. **nmbd を残す** → Tailnet IP には不要。masked
5. **home 共有で秘密が丸見え** → 常用 Documents
6. **Notion ポインタ skill に長文ミラーしない** → 本 skill。`raspberry-pi-tailscale-access` は到達性 Notion 正本

## Related

- `raspberry-pi-tailscale-access`
- `chromebook-hermes-home-mac-ssh`
- Host-Network Act（Pi `100.86.189.44` / Mac `100.127.122.8`）
