# Azure-DevOps-Problems-Solutions
Notes on Azure DevOps Problems and Notes

# 100 Azure DevOps Problems and Solutions

## Table of Contents
- [Pipelines](#pipelines)
- [Repositories](#repositories)
- [Boards](#boards)
- [Artifacts](#artifacts)
- [Test Plans](#test-plans)
- [Security and Access Control](#security-# 100 Azure DevOps Problems and Solutions

## Table of Contents
- [Pipelines](#pipelines)
- [Repositories](#repositories)
- [Boards](#boards)
- [Artifacts](#artifacts)
- [Test Plans](#test-plans)
- [Security and Access Control](#security-and-access-control)
- [Integration Issues](#integration-issues)
- [Performance](#performance)
- [Administration](#administration)
- [Migration](#migration)

## Pipelines

### 1. Pipeline fails with "Could not find a default agent pool"
**Problem:** When creating a new pipeline, it fails with an error about not finding a default agent pool.  
**Solution:** Either select an existing agent pool in your pipeline YAML using `pool: YourPoolName` or create a new agent pool in Project Settings > Pipelines > Agent Pools before running the pipeline.

### 2. "The pipeline is not valid. Job build: Step Docker needs a valid container reference"
**Problem:** Docker task in the pipeline fails with container reference error.  
**Solution:** Ensure your Docker task has a properly formatted container reference. Check that the Docker registry, repository, and tag are correctly specified in the pipeline YAML.

### 3. Variables not being passed between pipeline stages
**Problem:** Variables set in one stage are not accessible in another stage.  
**Solution:** Use output variables with proper syntax:
```yaml
- task: PowerShell@2
  name: SetVariableTask
  inputs:
    targetType: 'inline'
    script: |
      Write-Host "##vso[task.setvariable variable=myVar;isOutput=true]myValue"
```
And reference it in another stage:
```yaml
variables:
  myVar: $[ stageDependencies.previousStageName.SetVariableTask.outputs['SetVariableTask.myVar'] ]
```

### 4. "No hosted parallelism has been purchased or granted"
**Problem:** Pipeline fails with error about no parallelism being purchased.  
**Solution:** Your organization has exhausted its parallel job limits. Either wait for running pipelines to complete, purchase additional parallel jobs from Azure DevOps, or run at a time when fewer pipelines are active.

### 5. Unable to access secure files in pipeline
**Problem:** Pipeline cannot access secure files stored in the library.  
**Solution:** Make sure you're using the appropriate task:
```yaml
- task: DownloadSecureFile@1
  name: mySecureFile
  inputs:
    secureFile: 'myFile.pfx'
```
Then refer to it using `$(mySecureFile.secureFilePath)` in subsequent tasks.

### 6. "No such file or directory" errors when using script tasks
**Problem:** Scripts fail with file not found errors even though the files exist in the repo.  
**Solution:** Check the working directory of your script task. Add a `workingDirectory` setting or use `cd` commands to navigate to the correct directory before accessing files.

### 7. Pipeline fails with "Could not find or load main class"
**Problem:** Java builds fail with class not found errors.  
**Solution:** Verify your Java installation on the build agent and ensure the correct Java version is being used. Add a task to install the required Java version:
```yaml
- task: JavaToolInstaller@0
  inputs:
    versionSpec: '11'
    jdkArchitectureOption: 'x64'
    jdkSourceOption: 'PreInstalled'
```

### 8. PowerShell scripts fail in pipelines but work locally
**Problem:** PowerShell scripts run fine locally but fail in the pipeline.  
**Solution:** Check for execution policy settings or environment differences. In your pipeline, use:
```yaml
- task: PowerShell@2
  inputs:
    targetType: 'inline'
    script: 'Set-ExecutionPolicy -ExecutionPolicy Bypass -Scope Process -Force'
```
before running your PowerShell scripts.

### 9. "TF401019: The Git repository with name or identifier X does not exist"
**Problem:** Pipeline fails when trying to access a Git repository.  
**Solution:** Check that the repository exists and that the service connection or build agent has access permissions to it. Verify the repository name in your pipeline YAML.

### 10. Conditional expressions not working in pipelines
**Problem:** Pipeline conditions like `condition: and(succeeded(), eq(variables['Build.SourceBranch'], 'refs/heads/main'))` not working as expected.  
**Solution:** Double-check your syntax and use variables exactly as they appear in documentation. For branch filtering, consider using simpler triggers:
```yaml
trigger:
  branches:
    include:
    - main
    - releases/*
```

### 11. "Failed to resolve Docker endpoint with error: Docker endpoint does not exist"
**Problem:** Docker tasks fail with endpoint resolution errors.  
**Solution:** Create a service connection to your Docker registry in Project Settings > Service connections, then reference it in your Docker task:
```yaml
- task: Docker@2
  inputs:
    containerRegistry: 'your-registry-connection'
    repository: 'your-repository'
    command: 'buildAndPush'
```

### 12. "Error: Failed to get dependencies for NuGet package"
**Problem:** NuGet restore tasks fail in the pipeline.  
**Solution:** Add NuGet tool installer task before the restore step and configure authentication:
```yaml
- task: NuGetToolInstaller@1
  inputs:
    versionSpec: '5.x'

- task: NuGetCommand@2
  inputs:
    command: 'restore'
    restoreSolution: '**/*.sln'
    feedsToUse: 'select'
```

### 13. Cannot access private GitHub repositories in pipeline
**Problem:** Pipeline fails when trying to access a private GitHub repository.  
**Solution:** Create a GitHub service connection with appropriate access tokens in Project Settings > Service connections, then use it in your pipeline:
```yaml
resources:
  repositories:
  - repository: MyGitHubRepo
    type: github
    endpoint: MyGitHubConnection
    name: owner/repo
```

### 14. "NPM authentication error" when using private npm registry
**Problem:** npm install fails with authentication errors.  
**Solution:** Set up a service connection to your npm registry and use the npm authenticate task:
```yaml
- task: npmAuthenticate@0
  inputs:
    workingFile: '.npmrc'
    customEndpoint: 'YourNpmServiceConnection'

- script: npm install
```

### 15. Pipeline agent can't find a tool that's installed
**Problem:** Tools like Node.js, Python, or .NET SDK that are installed on the agent can't be found.  
**Solution:** Use the appropriate tool installer task before using the tool:
```yaml
- task: NodeTool@0
  inputs:
    versionSpec: '16.x'
    
- script: |
    node --version
    npm --version
```

### 16. Storage account access issues in release pipelines
**Problem:** Release pipeline fails when trying to access Azure Storage account.  
**Solution:** Create an Azure service connection with the right permissions, then use it in your Azure File Copy task:
```yaml
- task: AzureFileCopy@4
  inputs:
    SourcePath: '$(Build.ArtifactStagingDirectory)/*.zip'
    azureSubscription: 'YourAzureServiceConnection'
    Destination: 'AzureBlob'
    storage: 'storageaccountname'
    ContainerName: 'containername'
```

### 17. "TF400898: An Internal Error Occurred" in pipelines
**Problem:** Non-specific internal error occurs during pipeline execution.  
**Solution:** Check your organization's service health in Azure DevOps. If service is healthy, try cleaning agent workspace with `clean: true` in your YAML pool configuration or rebuilding from scratch.

### 18. Template expressions not getting evaluated
**Problem:** Variables or expressions like `${{ parameters.myParam }}` are not being evaluated in pipeline YAML.  
**Solution:** Check your syntax and make sure you're using the correct context. For runtime variables, use `$(variableName)`, for template expressions use `${{ expression }}`, and for runtime expressions use `$[ expression ]`.

### 19. "The hosted agent did not have required software"
**Problem:** The Microsoft-hosted agent is missing required software for your build.  
**Solution:** Use tool installer tasks to install necessary software or consider using a self-hosted agent where you can pre-install all required software.

### 20. YAML pipeline not picking up changes to the YAML file
**Problem:** Changes to pipeline YAML file are not being reflected in new pipeline runs.  
**Solution:** Make sure your changes are committed and pushed. If the problem persists, try editing the pipeline in the web editor and saving it, which forces Azure DevOps to refresh its cache.

### 21. Pipeline cancellation not working immediately
**Problem:** Pipeline doesn't stop immediately when canceled.  
**Solution:** This is expected behavior. The cancellation signal may take time to reach agents, especially if they're busy. For critical scenarios, configure shorter job timeouts.

### 22. "No space left on device" in pipeline agents
**Problem:** Pipeline fails with disk space errors on agents.  
**Solution:** For Microsoft-hosted agents, clean up workspace during the pipeline:
```yaml
steps:
- script: |
    df -h
    rm -rf node_modules/
    npm cache clean --force
    df -h
  displayName: 'Clean up disk space'
```
For self-hosted agents, increase disk space or set up regular maintenance.

### 23. Pipeline agent can't connect to external services
**Problem:** Pipeline fails when trying to connect to external services.  
**Solution:** Check if your agent needs to use a proxy to access external resources. Configure proxy settings in the agent configuration or use appropriate environment variables in your pipeline tasks.

### 24. Pipeline stuck in "Loading..." or "Waiting for agent" state
**Problem:** Pipeline appears to be stuck without progress.  
**Solution:** Check if there are available agents in your pool. If using Microsoft-hosted agents, there might be a queue or service issues. For self-hosted agents, verify they're online and have capacity.

### 25. Deployment fails with certificate validation errors
**Problem:** SSL/TLS certificate validation fails during deployment.  
**Solution:** For testing environments, you can bypass certificate validation (not recommended for production):
```yaml
- script: |
    AZURE_CLI_DISABLE_CONNECTION_VERIFICATION=1 az deployment group create --resource-group myGroup --template-file template.json
  displayName: 'Deploy with certificate validation disabled'
```
For production, ensure proper certificates are installed and trusted.

### 26. Timeouts during package restore or artifact download
**Problem:** Pipeline tasks that download packages or artifacts time out.  
**Solution:** Increase the timeout value for the specific task and consider caching to speed up subsequent runs:
```yaml
- task: NuGetCommand@2
  inputs:
    command: 'restore'
    restoreSolution: '**/*.sln'
    feedsToUse: 'select'
  timeoutInMinutes: 20
  
- task: Cache@2
  inputs:
    key: 'nuget | "$(Agent.OS)" | **/packages.lock.json,**/packages.config'
    path: '$(NUGET_PACKAGES)'
```

### 27. Caching not working between pipeline runs
**Problem:** Cache task doesn't seem to persist cache between pipeline runs.  
**Solution:** Verify your cache key is consistent between runs and that you're using the correct path. Also check that your cache size isn't exceeding limits:
```yaml
- task: Cache@2
  inputs:
    key: 'npm | "$(Agent.OS)" | package-lock.json'
    restoreKeys: |
      npm | "$(Agent.OS)"
    path: '$(npm_config_cache)'
  displayName: 'Cache npm packages'
```

### 28. Running out of build minutes
**Problem:** Organization runs out of free build minutes for Microsoft-hosted agents.  
**Solution:** Consider setting up self-hosted agents, optimizing pipelines to run faster, or purchasing additional parallel jobs and build minutes from Microsoft.

### 29. Pipeline environment approvals not working
**Problem:** Pipeline doesn't stop for approval even though approvals are configured.  
**Solution:** Make sure you're deploying to the environment using the correct syntax:
```yaml
stages:
- stage: Deploy
  jobs:
  - deployment: DeployWeb
    environment: 'Production'  # This environment should have approvals configured
    strategy:
      runOnce:
        deploy:
          steps:
          - script: echo Hello world!
```

### 30. Cannot run UI tests in pipelines
**Problem:** UI tests fail in pipelines due to lack of display or browser drivers.  
**Solution:** For Microsoft-hosted agents, use the appropriate task that supports UI testing or configure headless testing:
```yaml
- task: VSTest@2
  inputs:
    testSelector: 'testAssemblies'
    testAssemblyVer2: '**\*Tests.dll'
    runSettingsFile: 'headless.runsettings'
```

## Repositories

### 31. "git pull" command fails with "cannot lock ref" error
**Problem:** Git operations fail with lock errors in Azure Repos.  
**Solution:** This usually happens when multiple processes try to update references simultaneously. Try clearing Git credential cache:
```
git credential-manager uninstall
git credential-manager install
```

### 32. Files show as modified without changes in Git
**Problem:** Files appear as modified in Git status even though no content was changed.  
**Solution:** This is often due to line ending differences. Configure Git to handle line endings consistently:
```
git config --global core.autocrlf true  # for Windows
git config --global core.autocrlf input  # for Mac/Linux
```

### 33. "You are not authorized to access this repository" error
**Problem:** Unable to access a repository despite having project access.  
**Solution:** Check repository-specific permissions in Project Settings > Repositories > Your Repository > Security. Ensure your user or group has at least "Read" access.

### 34. Cannot delete a branch in Azure Repos
**Problem:** Getting "permission denied" when trying to delete a branch.  
**Solution:** Check if the branch has branch policies applied or if it's protected. Remove branch policies and protection or request a user with higher permissions to delete it.

### 35. Pull request blocked by policy but no clear error
**Problem:** Pull request cannot be completed due to policy violations without a clear message.  
**Solution:** Check all configured branch policies in Project Settings > Repositories > Branches > Select branch > Branch Policies. Common issues include required work items, reviewers, or failing builds.

### 36. Merge conflicts in pull requests
**Problem:** Frequent merge conflicts when trying to complete pull requests.  
**Solution:** Keep your branch updated with the target branch by regularly merging or rebasing:
```
# From your feature branch
git fetch origin
git merge origin/main

# Or with rebase
git fetch origin
git rebase origin/main
```

### 37. Large binary files causing repository performance issues
**Problem:** Repository becoming slow due to large binary files.  
**Solution:** Use Git LFS (Large File Storage) for binary files:
```
git lfs install
git lfs track "*.psd"
git lfs track "*.zip"
git add .gitattributes
```

### 38. "Error: failed to push some refs" when pushing to repository
**Problem:** Unable to push changes to the repository.  
**Solution:** This usually happens when the remote has changes you don't have locally. Pull first, then push:
```
git pull --rebase origin main
git push origin main
```

### 39. Cannot find deleted branch history
**Problem:** After deleting a branch, unable to find its commit history.  
**Solution:** Use the Git command line to find the commit history even for deleted branches:
```
git log --all --graph --decorate --oneline
```
Or check the "Pushes" tab in the repository's history view in Azure Repos.

### 40. Repository size growing too large
**Problem:** Git repository becoming too large and slow.  
**Solution:** Clean up large files and compress history:
```
# Find large files
git rev-list --objects --all | grep -f <(git verify-pack -v .git/objects/pack/*.idx | sort -k 3 -n | tail -10 | awk '{print $1}')

# Use BFG Repo Cleaner to remove large files from history
```
Consider breaking up large repositories into smaller ones or using Git LFS.

## Boards

### 41. Work items disappear from backlog or boards
**Problem:** Work items that should be visible are missing from backlog or boards.  
**Solution:** Check your filter settings and ensure the correct iteration and area paths are selected. Also verify that the work item state matches what your current view is configured to show.

### 42. Sprint capacity planning not working correctly
**Problem:** Team capacity calculations seem incorrect in sprint planning.  
**Solution:** Make sure all team members have their capacity set correctly, including days off. Navigate to Boards > Sprints > Capacity and update individual capacity and days off.

### 43. Cannot customize process for work items
**Problem:** Unable to modify work item forms or add custom fields.  
**Solution:** Check if you're using an inherited process or a default process. Only inherited processes can be customized. Go to Organization Settings > Process > Create an inherited process from the one you're using, then customize it.

### 44. Work item queries returning unexpected results
**Problem:** Work item queries don't return the expected items.  
**Solution:** Review query clauses carefully, especially when using AND/OR operators. Use the query editor's "Group clauses" feature to ensure proper logical grouping. Preview results frequently when building complex queries.

### 45. Unable to assign work items to certain users
**Problem:** Cannot assign work items to specific users who should have access.  
**Solution:** Verify that the users are members of the project team and have the appropriate access level. Check if there are any security restrictions on work item assignments in Project Settings > Security.

### 46. Missing data in burndown charts
**Problem:** Sprint burndown charts show incomplete or missing data.  
**Solution:** Ensure work items have proper estimates in the "Effort," "Story Points," or equivalent field. Check that the Sprint start and end dates are set correctly, and that work items are correctly assigned to the Sprint.

### 47. Board columns not reflecting work item state changes
**Problem:** Changing a work item's state doesn't move it to the expected column on the board.  
**Solution:** Review your board column configuration. Go to Board settings > Columns and check the mapping between work item states and board columns.

### 48. Cannot move items in rank order on the backlog
**Problem:** Unable to reorder items by dragging them in the backlog view.  
**Solution:** Check if there are active filters applied to the backlog. Filtering can disable reordering. Also, ensure you have the necessary permissions to reorder work items.

### 49. Unable to link work items to pull requests or commits
**Problem:** Option to link work items to code artifacts is missing or not working.  
**Solution:** Make sure the repository and work item are in the same project or that cross-project linking is enabled. Use the # syntax in commit messages or PR descriptions (e.g., "Fixes #123").

### 50. Team members cannot see all work items
**Problem:** Some team members cannot see certain work items that others can.  
**Solution:** Check area path permissions. Configure them in Project Settings > Project configuration > Areas > Security. Ensure teams have access to the appropriate area paths.

### 51. Work item field rules not triggering correctly
**Problem:** Custom field rules (like required fields or conditional rules) don't work as expected.  
**Solution:** In your process customization, review rule conditions and actions. Make sure rules are applied to the correct process and work item types. Test with simpler rules first to isolate issues.

### 52. Sprint dates incorrect or not showing
**Problem:** Sprint dates appear incorrect or sprints are not visible in planning tools.  
**Solution:** Check project settings to ensure sprint dates are configured correctly. Go to Project Settings > Project configuration > Iterations to set or correct sprint dates.

### 53. Board swimlanes not separating items correctly
**Problem:** Work items are not appearing in the expected swimlanes.  
**Solution:** Check your swimlane configuration at Board settings > Swimlanes. Verify that the rules for swimlane assignment match your expectations.

## Artifacts

### 54. "401 Unauthorized" when accessing Azure Artifacts feeds
**Problem:** Unable to access package feeds with 401 errors.  
**Solution:** Generate and use a Personal Access Token (PAT) with the appropriate scopes:
```
npm config set registry https://pkgs.dev.azure.com/your-org/_packaging/your-feed/npm/registry/
npm config set //pkgs.dev.azure.com/your-org/_packaging/your-feed/npm/registry/:_authToken=YOUR_PAT
```

### 55. Package version conflicts in Artifacts feeds
**Problem:** Unable to publish packages due to version conflicts.  
**Solution:** Azure Artifacts doesn't allow overwriting existing package versions. Use a new version number or delete the existing package version (if allowed by retention policies).

### 56. Slow package download from Artifacts feeds
**Problem:** Package restores are slow when using Azure Artifacts.  
**Solution:** Consider using upstream sources and caching to improve performance. Configure your package manager to use the cache:
```
# For npm
npm config set cache /path/to/npm-cache --global

# For NuGet
nuget config -set globalPackagesFolder=/path/to/nuget-cache
```

### 57. Unable to delete packages from feeds
**Problem:** Cannot delete packages even with administrator permissions.  
**Solution:** Check if the package is in use or if there are feed retention policies preventing deletion. If necessary, update feed permissions in Feed settings > Permissions and add yourself as an Owner.

### 58. "Package not found" errors despite the package existing
**Problem:** Package manager can't find packages that are visible in the Azure Artifacts UI.  
**Solution:** Check that your package source URLs are correct and that authentication is properly configured. For Maven, check your `settings.xml`; for NuGet, check your `nuget.config`.

### 59. Artifacts retention policy deleting needed packages
**Problem:** Important packages are being automatically deleted by retention policies.  
**Solution:** Mark important package versions as "Preserved" in the Azure Artifacts UI, which exempts them from retention policies.

### 60. Cannot connect to feeds from CI/CD pipelines
**Problem:** Pipeline fails to authenticate with Azure Artifacts feeds.  
**Solution:** Use built-in authentication for pipelines:
```yaml
steps:
- task: NuGetAuthenticate@0
  displayName: 'NuGet Authenticate'
  
- script: dotnet restore
  displayName: 'Restore packages'
```

## Test Plans

### 61. Test plans not appearing in Test Plans hub
**Problem:** Unable to see existing test plans in the Test Plans interface.  
**Solution:** Check that you have the necessary test plan license and permissions. Also, ensure you're looking in the correct project and iteration.

### 62. Cannot add test cases to test suites
**Problem:** Option to add test cases to test suites is missing or disabled.  
**Solution:** Verify that you have the "Create Test Cases" permission and that the test plan is not in a completed state. For hierarchy-based test suites, ensure the requirement work items exist.

### 63. Manual test results not being recorded
**Problem:** Results of manual tests are not being saved.  
**Solution:** Make sure you're using the Test Runner properly by clicking "Save and Close" after each test. Check browser extensions or popup blockers that might interfere with the Test Runner.

### 64. Cannot assign test cases to testers
**Problem:** Unable to assign test cases to specific team members.  
**Solution:** Verify that the users have the appropriate license and project permissions. Assign tests through the Test Plan grid view by selecting tests and using the "Assign to" option.

### 65. Test attachments missing or corrupted
**Problem:** Files attached to test results are missing or cannot be opened.  
**Solution:** Check attachment size limits (usually 60MB). For larger files, consider compressing them or using external storage with links.

### 66. Cannot create or modify test configurations
**Problem:** Unable to add or edit test configurations for different test environments.  
**Solution:** Check if you have the "Manage test configurations" permission. Create configurations in Project Settings > Test > Configurations.

### 67. Unable to export test plans or results
**Problem:** Export functionality for test plans or results is not working.  
**Solution:** Use the built-in Excel integration or consider using the Azure DevOps REST API to extract test data programmatically.

### 68. Test parameters not being applied during test runs
**Problem:** Parameterized tests don't receive the expected parameter values.  
**Solution:** Verify that parameters are correctly defined in the test case and that parameter values are set in the test configuration. Review the test case steps to ensure they reference parameters correctly using `@parameter`.

## Security and Access Control

### 69. "TF401019: The user is not authorized to access X" despite having permissions
**Problem:** Users get access denied errors even though they appear to have the necessary permissions.  
**Solution:** Check for inherited permissions and permission conflicts. Look at both explicit permissions and permissions inherited from groups. Consider removing all explicit denies first.

### 70. Cannot add users to Azure DevOps organization
**Problem:** Unable to add certain users to the organization.  
**Solution:** Verify that your Azure Active Directory settings allow the users to be added. Check for user cap limits in your organization and consider removing inactive users.

### 71. Service connections failing with authentication errors
**Problem:** Service connections to external services fail with authentication errors.  
**Solution:** Regenerate service connection credentials and verify that the service principal or user account has the necessary permissions in the target service. For Azure connections, check service principal expiration dates.

### 72. PAT tokens expire unexpectedly
**Problem:** Personal Access Tokens stop working before their expiration date.  
**Solution:** PATs can be revoked by administrators or security policies. Create new tokens with appropriate scopes and longer expiration periods. Consider using service principals for automated processes instead of PATs.

### 73. "Forbidden" errors when accessing certain features
**Problem:** Users unable to access specific features despite having project access.  
**Solution:** Check feature-specific permissions in Project Settings > Security. Different features (Repos, Pipelines, Boards) have their own permission sets that must be configured.

### 74. IP restrictions preventing access
**Problem:** Some users cannot access Azure DevOps due to IP restrictions.  
**Solution:** Review organization security policies in Organization Settings > Security > Policies. Adjust IP allow lists or VPN requirements as needed.

### 75. Secret values visible in pipeline logs
**Problem:** Sensitive values like passwords appear in plain text in pipeline logs.  
**Solution:** Use secret variables and environment variables for sensitive information:
```yaml
variables:
  - group: 'my-secret-variables'
  
steps:
- script: |
    echo Using secret without displaying it: $(secretVariable)
  env:
    secretVariable: $(MY_SECRET)
```

### 76. Cross-project access issues
**Problem:** Users or pipelines cannot access resources across projects in the same organization.  
**Solution:** Configure cross-project permissions explicitly. For pipelines, use service connections or project-scoped build service identities with appropriate permissions.

## Integration Issues

### 77. Azure DevOps-GitHub integration not working
**Problem:** Unable to connect Azure DevOps to GitHub repositories.  
**Solution:** Re-create your GitHub service connection with the appropriate OAuth scopes or PAT permissions. Ensure the GitHub account has access to the repositories you're trying to connect to.

### 78. Microsoft Teams notifications not being delivered
**Problem:** Azure DevOps notifications not appearing in Microsoft Teams channels.  
**Solution:** Verify the Teams connector configuration. Try removing and re-adding the connection, ensuring you have permissions in both systems. Test with a simple notification to isolate the issue.

### 79. SMTP/Email notifications not being sent
**Problem:** Email alerts for work item changes or build completions not being delivered.  
**Solution:** Check organization notification settings in Organization Settings > Notifications. Verify SMTP configuration and ensure email addresses are correct and not blocked.

### 80. Third-party tool integration stops working after Azure DevOps update
**Problem:** Integrations with external tools break after Azure DevOps platform updates.  
**Solution:** Check for version compatibility issues and update the integration's connection settings. Review the Azure DevOps release notes for API changes that might affect integrations.

### 81. Unable to connect to Excel for work item updates
**Problem:** Excel integration not working for bulk work item updates.  
**Solution:** Install the latest Azure DevOps Office Integration plugin. Ensure you're using a compatible version of Excel and that your PAT has the correct scopes.

### 82. Power BI reports not refreshing with latest data
**Problem:** Power BI reports connected to Azure DevOps analytics not showing current data.  
**Solution:** Check that Analytics views are properly configured and refreshing. Set appropriate refresh schedules in Power BI and verify service account permissions.

### 83. Azure DevOps extensions not loading
**Problem:** Marketplace extensions fail to load or work properly after installation.  
**Solution:** Clear browser cache and cookies. Verify that the extension is compatible with your Azure DevOps version and that it has the necessary permissions.

## Performance

### 84. Azure DevOps web interface is slow or unresponsive
**Problem:** Web portal takes too long to load pages or respond to actions.  
**Solution:** Check your network connection and try a different browser. Clear browser cache and cookies. If the issue persists, check Azure DevOps service health or consider using a different region.

### 85. Query execution timeouts
**Problem:** Complex work item queries fail with timeout errors.  
**Solution:** Simplify queries by breaking them into smaller ones. Avoid using too many "OR" conditions, and limit the use of recursive path-based queries when possible.

### 86. Slow Git clone or fetch operations
**Problem:** Git operations taking too long, especially for large repositories.  
**Solution:** Use shallow clones for build pipelines:
```
git clone --depth 1 --branch main <repository-url>
```
For regular use, consider using Git sparse checkout or splitting large repositories.

### 87. Dashboard widgets loading slowly
**Problem:** Custom dashboard widgets take too long to load.  
**Solution:** Reduce the number of widgets on a single dashboard. For query-based widgets, simplify the underlying queries. Consider creating multiple focused dashboards instead of one complex dashboard.

### 88. Work item form takes too long to load
**Problem:** Opening individual work items in the web interface is very slow.  
**Solution:** Simplify custom work item forms by removing unnecessary controls. Reduce the number of linked items and attachments on frequently used work items.

### 89. Build pipeline queue times are excessive
**Problem:** Builds sit in the queue for too long before starting.  
**Solution:** Increase the number of parallel jobs or agents available. Optimize build pipelines to run faster. Consider self-hosted agents for more capacity.

## Administration

### 90. Unable to change organization owner
**Problem:** Cannot transfer organization ownership to another user.  
**Solution:** The new owner must be in the Project Collection Administrators group and must have logged in to the organization at least once. Use the Change Owner option in Organization Settings.

### 91. Organization billing issues
**Problem:** Unable to update billing information or purchase additional services.  
**Solution:** Verify that you have Owner or Admin permissions on both the Azure DevOps organization and the linked Azure subscription. Check if there are any spending limits or policies on the Azure subscription.

### 92. Process customization changes not applying
**Problem:** Changes to inherited processes don't appear in projects using those processes.  
**Solution:** Process changes are not retroactive for existing work items. Create a new work item to see the changes, or use the Process Migration tool to update existing items.

### 93. Unable to rename or move a project
**Problem:** Options to rename or move a project are disabled or fail with errors.  
**Solution:** Verify you have Project Collection Administrator permissions. Stop all running pipelines and active operations before attempting to rename or move a project.

### 94. Cannot delete a project
**Problem:** Project deletion fails or the option is not available.  
**Solution:** Check that you have Project Collection Administrator permissions. Ensure all resources within the project (like build definitions and release environments) are removed first. Try using the Azure DevOps CLI for deletion.

### 95. Organization restore from deletion fails
**Problem:** Unable to restore a recently deleted organization.  
**Solution:** Deleted organizations can only be restored within 28 days and only by the original deleter. Contact Azure DevOps support if you're within this window but unable to restore.

## Migration

### 96. Migration from TFS/Azure DevOps Server fails
**Problem:** Data migration from on-premises TFS or Azure DevOps Server to Azure DevOps Services fails.  
**Solution:** Use the official Data Migration Tool and follow Microsoft's migration guide carefully. Ensure your source and target versions are compatible and that you have the necessary permissions in both systems.

### 97. Work item attachments missing after migration
**Problem:** File attachments on work items are missing after migrating to Azure DevOps.  
**Solution:** Check attachment size limits and migration logs. Reattach critical files manually or use a script to download from the source system and reattach to the target system.

### 98. Test result history lost during migration
**Problem:** Historical test results aren't visible after migration.  
**Solution:** Test results often don't migrate completely. Consider keeping the old system available for historical reference or export important test data before migration.

### 99. Source code history incomplete after migration
**Problem:** Git repository history appears incomplete after migration.  
**Solution:** Ensure you performed a full clone with all branches and tags before pushing to the new repository. Use `git clone --mirror` for the initial clone to preserve all references.

### 100. Custom fields and processes not migrated correctly
**Problem:** Custom work item fields, states, or processes are missing or incorrect after migration.  
**Solution:** Process customizations often require manual recreation in the target system. Document all customizations before migration and verify each one after migration is complete. Use the Process Template editor to recreate custom fields and rules.
