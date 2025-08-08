# 2025 Desktop DSC Packages CI Workflow

This repository contains an automated GitHub Actions workflow to install a collection of desktop software packages on a Windows runner using PowerShell DSC (Desired State Configuration) and Chocolatey.

## Workflow Overview

The workflow is defined in `.github/workflows/2025-desktop-dsc-packages-ci.yml`. It is triggered by two events:

1.  A `push` to the `main` branch.
2.  A scheduled cron job that runs once a day at 11:39 AM UTC.

The workflow's primary goal is to ensure a consistent and repeatable software installation process.

## How the Workflow Works

The workflow consists of a single job named `"2025 DSC packages"` that runs on a `windows-2025` runner. Each step in the job is critical to the process:

1.  **Checkout**: Fetches the code from the repository.
2.  **Install DSC Modules**: Installs the required PowerShell DSC modules (`PSDscResources` and `cChoco`) from the PowerShell Gallery. This step is crucial as it provides the resources needed to manage packages via Chocolatey.
3.  **Compile DSC Configuration**: This step executes the PowerShell DSC script (`scripts/Apply_DSC_Desktop_Config.ps1`). It performs two key actions:
    * **Dot-sourcing**: It loads the `Configuration` block into the PowerShell session.
    * **Compilation**: It invokes the `DesktopPackages` configuration function, which compiles the desired state into a `.mof` file and places it in a directory named `DesktopPackages`.
4.  **Apply DSC Configuration**: This is where the installation happens. The `Start-DscConfiguration` cmdlet reads the compiled `.mof` file from the `DesktopPackages` directory and enforces the desired state, installing all the specified packages using Chocolatey.
5.  **Verify Installations**: A crucial final step that confirms the success of the installation. It runs a series of simple commands (`git --version`, `Get-Command`) to verify that a few key packages were installed and are accessible in the system's path.

## PowerShell DSC Configuration (`Apply_DSC_Desktop_Config.ps1`)

The core logic for package installation is defined in the `Apply_DSC_Desktop_Config.ps1` script. This script declares a `Configuration DesktopPackages` block, which:

-   Ensures Chocolatey itself is installed using the `cChocoInstaller` resource.
-   Uses the `cChocoPackageInstallerSet` resource to define a desired state for various package groups (e.g., `System Tools`, `Browsers`, `Development`). The script tells Chocolatey which packages to install.

This approach is highly declarative, meaning you describe the end state you want, and DSC handles the steps to get there.

## Customization

To customize the list of installed software:
1.  Open the `scripts/Apply_DSC_Desktop_Config.ps1` file.
2.  Modify the `Name` arrays within the `cChocoPackageInstallerSet` resource blocks to add or remove package names.
3.  Commit your changes and push them to the `test` branch to trigger the workflow.

This declarative approach makes it simple to maintain a consistent set of desktop applications across your CI/CD environment.