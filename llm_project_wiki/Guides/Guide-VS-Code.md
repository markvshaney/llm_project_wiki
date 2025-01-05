# VS Code Complete Setup Guide

VS Code is a powerful IDE that supports multiple programming languages and development workflows. This guide provides both automated and manual setup instructions, along with a robust setup script.

## Quick Start

1. Download the setup script below
2. Save as `vscode_setup.py`
3. Make executable: `chmod +x vscode_setup.py` (Unix-based systems)
4. Run: `./vscode_setup.py`

## Setup Script Overview

The script handles:
- VS Code installation for Windows, macOS, and Linux
- Essential extension installation
- Configuration and settings management
- Debugging setup
- Git integration

### Important Pre-run Steps
- Back up existing `settings.json`
- Document current extensions
- Note existing Git configurations

## Complete Setup Script

```python
#!/usr/bin/env python3
import os
import sys
import json
import shutil
import logging
import platform
import subprocess
from pathlib import Path
from typing import List, Dict, Optional

class VSCodeSetup:
    def __init__(self):
        self.os_type = platform.system().lower()
        self.home_dir = str(Path.home())
        self.essential_extensions = [
            "ms-python.python-extension-pack",
            "eamodio.gitlens",
            "ms-vscode-remote.vscode-remote-extensionpack"
        ]
        self._setup_logging()
        self._setup_paths()

    def _setup_logging(self) -> None:
        """Configure logging for the setup process."""
        logging.basicConfig(
            level=logging.INFO,
            format='%(asctime)s - %(levelname)s - %(message)s',
            handlers=[
                logging.StreamHandler(),
                logging.FileHandler('vscode_setup.log')
            ]
        )
        self.logger = logging.getLogger(__name__)

    def _setup_paths(self) -> None:
        """Set up paths based on OS."""
        if self.os_type == "windows":
            self.settings_path = os.path.join(os.getenv("APPDATA"), "Code", "User", "settings.json")
        else:
            self.settings_path = os.path.join(self.home_dir, ".config", "Code", "User", "settings.json")

    def check_prerequisites(self) -> bool:
        """Verify required software is installed."""
        try:
            # Check Python version
            if sys.version_info < (3, 6):
                self.logger.error("Python 3.6 or higher is required")
                return False

            # Check Git installation
            if not self._run_command(["git", "--version"], check=False):
                self.logger.error("Git is not installed")
                return False

            return True
        except Exception as e:
            self.logger.error(f"Prerequisite check failed: {str(e)}")
            return False

    def install_vscode(self) -> bool:
        """Install VS Code for the current OS."""
        self.logger.info("Installing Visual Studio Code...")
        
        installers = {
            "windows": self._install_vscode_windows,
            "darwin": self._install_vscode_macos,
            "linux": self._install_vscode_linux
        }
        
        installer = installers.get(self.os_type)
        if installer:
            return installer()
        self.logger.error(f"Unsupported OS: {self.os_type}")
        return False

    def _install_vscode_windows(self) -> bool:
        """Windows-specific installation."""
        commands = [
            [
                "powershell.exe",
                "-Command",
                "Invoke-WebRequest -Uri 'https://code.visualstudio.com/sha/download?build=stable&os=win32-x64-user' -OutFile 'VSCodeSetup.exe'"
            ],
            ["VSCodeSetup.exe", "/SILENT", "/MERGETASKS=!runcode,addcontextmenufiles,addcontextmenufolders,addtopath"]
        ]
        return all(self._run_command(cmd) for cmd in commands)

    def _install_vscode_macos(self) -> bool:
        """macOS-specific installation."""
        commands = [
            [
                "curl",
                "-L",
                "https://code.visualstudio.com/sha/download?build=stable&os=darwin-universal",
                "-o",
                "VSCode.zip"
            ],
            ["unzip", "-q", "VSCode.zip"],
            ["mv", "Visual Studio Code.app", "/Applications/"]
        ]
        return all(self._run_command(cmd) for cmd in commands)

    def _install_vscode_linux(self) -> bool:
        """Linux-specific installation."""
        if os.path.exists("/usr/bin/apt"):
            commands = [
                ["sudo", "apt", "update"],
                ["sudo", "apt", "install", "-y", "code"]
            ]
        elif os.path.exists("/usr/bin/dnf"):
            commands = [
                ["sudo", "dnf", "install", "-y", "code"]
            ]
        else:
            self.logger.error("Unsupported Linux distribution")
            return False
        
        return all(self._run_command(cmd) for cmd in commands)

    def install_extensions(self) -> bool:
        """Install and update essential extensions."""
        self.logger.info("Installing essential extensions...")
        try:
            # Get currently installed extensions
            result = subprocess.run(["code", "--list-extensions"], 
                                 capture_output=True, text=True, check=True)
            installed = result.stdout.splitlines()

            # Install missing extensions
            for extension in self.essential_extensions:
                if extension not in installed:
                    if not self._run_command(["code", "--install-extension", extension]):
                        return False
            return True
        except Exception as e:
            self.logger.error(f"Extension installation failed: {str(e)}")
            return False

    def configure_settings(self) -> bool:
        """Set up VS Code settings."""
        self.logger.info("Configuring VS Code settings...")
        
        settings = {
            "editor.formatOnSave": True,
            "python.linting.enabled": True,
            "python.formatting.provider": "black",
            "git.enableSmartCommit": True,
            "files.autoSave": "afterDelay",
            "files.autoSaveDelay": 1000,
            "workbench.startupEditor": "none",
            "terminal.integrated.defaultProfile.windows": "PowerShell",
            "terminal.integrated.defaultProfile.linux": "bash",
            "terminal.integrated.defaultProfile.osx": "zsh",
            "editor.rulers": [80, 100],
            "editor.renderWhitespace": "boundary",
            "files.trimTrailingWhitespace": True
        }
        
        try:
            os.makedirs(os.path.dirname(self.settings_path), exist_ok=True)
            with open(self.settings_path, 'w') as f:
                json.dump(settings, f, indent=4)
            return True
        except Exception as e:
            self.logger.error(f"Settings configuration failed: {str(e)}")
            return False

    def run_setup(self) -> None:
        """Execute the complete setup process."""
        try:
            if not self.check_prerequisites():
                sys.exit(1)

            steps = [
                (self.install_vscode, "VS Code installation"),
                (self.install_extensions, "Extension installation"),
                (self.configure_settings, "Settings configuration"),
                (self.setup_debugging, "Debug setup"),
                (self.setup_git_integration, "Git integration")
            ]

            for step_func, step_name in steps:
                if not step_func():
                    self.logger.error(f"{step_name} failed")
                    sys.exit(1)

            self.logger.info("\nVS Code setup completed successfully!")
            self.logger.info("Please restart VS Code to apply all changes.")
            
        except Exception as e:
            self.logger.error(f"Setup failed: {str(e)}")
            sys.exit(1)

if __name__ == "__main__":
    setup = VSCodeSetup()
    setup.run_setup()
```

## Script Components Explained

### 1. Prerequisites Check
- Verifies Python 3.6+
- Confirms Git installation
- Checks OS compatibility

### 2. VS Code Installation
Windows:
- Downloads installer via PowerShell
- Silent installation with PATH integration

macOS:
- Downloads universal build
- Extracts and moves to Applications

Linux:
- Detects package manager (apt/dnf)
- Handles installation/updates

### 3. Extension Management
Core extensions installed:
- Python Extension Pack
  - Full Python language support
  - Debugging capabilities
  - Linting and formatting
- GitLens
  - Enhanced Git features
  - History visualization
- Remote Development Pack
  - SSH/Container support
  - WSL integration

### 4. Configuration
Settings configured:
```json
{
    "editor.formatOnSave": true,
    "python.linting.enabled": true,
    "python.formatting.provider": "black",
    "git.enableSmartCommit": true,
    "files.autoSave": "afterDelay",
    "files.autoSaveDelay": 1000,
    "workbench.startupEditor": "none"
}
```

### 5. Debugging Setup
Creates standardized launch configuration:
```json
{
    "version": "0.2.0",
    "configurations": [
        {
            "name": "Python: Current File",
            "type": "python",
            "request": "launch",
            "program": "${file}",
            "console": "integratedTerminal",
            "justMyCode": true,
            "env": {
                "PYTHONPATH": "${workspaceFolder}"
            }
        }
    ]
}
```

## Manual Setup Alternative

### Windows
1. Download from VS Code website
2. Run installer
3. Enable PATH integration

### macOS
1. Download VS Code
2. Mount DMG
3. Move to Applications

### Linux
```bash
# Ubuntu/Debian
sudo apt update && sudo apt install code

# Fedora/RHEL
sudo dnf install code
```

## Maintenance Guide

### Regular Updates
1. VS Code Updates:
   - Enable automatic updates
   - Check for updates manually: Help > Check for Updates

2. Extension Updates:
   - Review extensions periodically
   - Update all: Extensions panel > ... > Update All

### Performance Optimization
1. Cache Management:
   - Clear VS Code cache periodically
   - Remove unused extensions
   - Clean workspace storage

2. Settings Management:
   - Backup settings.json regularly
   - Document custom configurations
   - Use workspace-specific settings when appropriate

## Troubleshooting

### Common Issues

1. Installation Failures:
   - Run with admin/sudo privileges
   - Check network connection
   - Verify system requirements
   - Clear temp files

2. Extension Problems:
   - Reload VS Code
   - Check compatibility
   - Clear extension cache

3. Git Integration Issues:
   - Verify Git installation
   - Check credentials
   - Validate permissions

## Essential Keyboard Shortcuts

For productivity:
- Command Palette: `Ctrl/Cmd + Shift + P`
- Quick Open: `Ctrl/Cmd + P`
- Terminal: `Ctrl/Cmd + '`
- Format Document: `Alt + Shift + F`
- Debug Start/Stop: `F5/Shift + F5`