# Pull and restart on VPS

Run each command separately (one line at a time). Do not combine commands.

## Simple dev pull (separate commands)

```bash
cd /root/infra-TAK
git pull origin dev
sudo systemctl restart takwerx-console
```

## Dev branch (explicit flow)

```bash
cd /root/infra-TAK
git fetch origin
git checkout dev
git pull --ff-only origin dev
sudo systemctl restart takwerx-console
```

## Main branch (stable)

```bash
cd /root/infra-TAK
git fetch origin
git checkout main
git pull --ff-only origin main
sudo systemctl restart takwerx-console
```

*(Repo not in `/root/infra-TAK`? Replace with your actual path.)*

More: `docs/COMMANDS.md`
