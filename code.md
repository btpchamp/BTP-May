### Activity 1: Initialize Your First Node.js Project (10 minutes)

```bash
# Create project folder:
mkdir ~/cap-training/day9-project
cd ~/cap-training/day9-project

# Initialize with npm:
npm init -y

# Look at what was created:
cat package.json
```

Now install some packages:

```bash
# Install a date formatting library:
npm install dayjs

# Install a color library for console output:
npm install chalk@4

# Install a dev-only testing tool:
npm install --save-dev jest

# Check what's in your folder:
ls node_modules/    # See all installed packages
cat package.json    # See dependencies added
```
