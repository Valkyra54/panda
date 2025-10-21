#!/usr/bin/env bash
#
# commit_and_push.sh
# Cara pakai:
#   1. letakkan di root repo
#   2. chmod +x commit_and_push.sh
#   3. ./commit_and_push.sh "Pesan commit singkat" optional-path1 optional-path2...
#
# Jika remote belum ada, script akan meminta URL remote (origin).
#

set -euo pipefail

MSG="${1:-"Auto commit: $(date +"%Y-%m-%d %H:%M:%S")"}"
shift || true
PATHS=("$@")

# kalau tidak di dalam repo git, inisialisasi
if ! git rev-parse --is-inside-work-tree >/dev/null 2>&1; then
  echo "Tidak ada repo git. Menginisialisasi repository baru..."
  git init
fi

# set remote origin jika belum ada
if ! git remote get-url origin >/dev/null 2>&1; then
  read -rp "Remote 'origin' belum terpasang. Masukkan URL GitHub remote (contoh: git@github.com:user/repo.git): " REMOTE
  if [ -n "$REMOTE" ]; then
    git remote add origin "$REMOTE"
  fi
fi

# stage files (semua atau path yang diberikan)
if [ ${#PATHS[@]} -eq 0 ]; then
  git add -A
else
  git add "${PATHS[@]}"
fi

# commit hanya jika ada perubahan
if git diff --cached --quiet; then
  echo "Tidak ada perubahan untuk di-commit."
else
  git commit -m "$MSG"
  # pilih branch default (cek remote atau gunakan main)
  BRANCH=$(git rev-parse --abbrev-ref HEAD 2>/dev/null || echo "main")
  # coba push, buat upstream jika perlu
  if git rev-parse --abbrev-ref --symbolic-full-name @{u} >/dev/null 2>&1; then
    git push
  else
    echo "Mencoba push dan membuat upstream $BRANCH..."
    git push -u origin "$BRANCH"
  fi
  echo "Selesai: commit & push dengan pesan: $MSG"
fi
