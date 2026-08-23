##!/usr/bin/env bash
#
# setup_portfolio.sh
# Scaffolds a full local GitHub portfolio structure with README templates,
# .gitignore, and LICENSE for each project. Optionally creates and pushes
# the repos to GitHub automatically if the GitHub CLI (`gh`) is installed
# and authenticated (run `gh auth login` first).
#
# Usage:
#   chmod +x setup_portfolio.sh
#   ./setup_portfolio.sh
#
# Edit the PROJECTS array below to match your actual project names.

set -e

GITHUB_USERNAME="YOUR_GITHUB_USERNAME"   # <-- change this
BASE_DIR="$HOME/portfolio"
AUTO_PUSH=false   # set to true if you have `gh` installed and authenticated

# project_folder_name : Short description : visibility(public/private)
PROJECTS=(
  "bandit-writeups:OverTheWire Bandit walkthroughs (all 34 levels) - Linux fundamentals and privilege escalation basics:public"
  "packet-tracer-labs:Progressive Cisco Packet Tracer networking labs:public"
  "alten-global-network-simulation:Capstone project - multi-site WAN, VLANs, router-on-a-stick, ACLs, custom HTTP portal:public"
  "home-lab-notes:Home lab build log - GNS3/EVE-NG, pfSense, Security Onion, automation:public"
  "python-security-scripts:Security-focused Python scripts and automation tools:public"
)

mkdir -p "$BASE_DIR"
cd "$BASE_DIR"

echo "Setting up portfolio in $BASE_DIR ..."
echo ""

for entry in "${PROJECTS[@]}"; do
  IFS=":" read -r name desc visibility <<< "$entry"

  if [ -d "$name" ]; then
    echo "Skipping '$name' (already exists)"
    continue
  fi

  echo "Creating project: $name"
  mkdir -p "$name"
  cd "$name"

  # README template
  cat > README.md << EOF
# ${name//-/ }

$desc

## Overview

_Describe what this project covers and why you built it._

## What I learned

- _Point 1_
- _Point 2_
- _Point 3_

## Structure

\`\`\`
.
├── README.md
└── (add your files, notes, configs, scripts here)
\`\`\`

## Notes / Writeup

_Add your detailed writeup, screenshots, or links here._

---
Part of my [portfolio](https://github.com/$GITHUB_USERNAME).
EOF

  # .gitignore
  cat > .gitignore << 'EOF'
.DS_Store
*.log
__pycache__/
*.pyc
.env
.vscode/
.idea/
EOF

  # MIT License
  cat > LICENSE << EOF
MIT License

Copyright (c) $(date +%Y) $GITHUB_USERNAME

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT.
EOF

  git init -q
  git add .
  git commit -q -m "Initial commit: scaffold $name"

  echo "  -> local repo ready at $BASE_DIR/$name"

  if [ "$AUTO_PUSH" = true ] && command -v gh &> /dev/null; then
    gh repo create "$GITHUB_USERNAME/$name" --"$visibility" --source=. --remote=origin --push
    echo "  -> pushed to https://github.com/$GITHUB_USERNAME/$name"
  fi

  cd "$BASE_DIR"
  echo ""
done

echo "Done. Projects scaffolded in: $BASE_DIR"
echo ""
if [ "$AUTO_PUSH" = false ]; then
  echo "AUTO_PUSH is off. To push manually for each project:"
  echo "  cd $BASE_DIR/<project-name>"
  echo "  gh repo create $GITHUB_USERNAME/<project-name> --public --source=. --remote=origin --push"
  echo "(or create the repo on github.com and use: git remote add origin <url> && git push -u origin main)"
fi
