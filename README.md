name: CI Gradle Pipeline

env:
  SCA_TOOL: 'snyk'
  CONTAINER_SCAN_TOOL: 'snyk'
  SAST_TOOL: 'snyk'

on:
  workflow_call:
    inputs:
      RUNS_ON:
        required: false
        description: Determines which runner to run on
        type: string
        default: self-hosted
      JAVA_VERSION:
        required: false
        description: The version of the language to use. This is used to determine the build docker container version used.
        type: string
        default: "17.0.9"
      JAVA_TEST_COMMAND:
        required: false
        description: The command used to run unit tests. Set to “NA” to disable.
        type: string
        default: "./gradlew clean check"
      JAVA_LINT_COMMAND:
        required: false
        description: The command used to lint the code. Set to “NA” to disable.
        type: string
        default: "./gradlew lint"
      JAVA_DEPS_COMMAND:
        required: false
        description: The command used to download dependencies
        type: string
        default: "./gradlew --refresh-dependencies"
      JAVA_BUILD_COMMAND:
        required: false
        description: The command used to build the application
        type: string
        default: "./gradlew clean build"
      WORKSPACE:
        required: false
        default: ${{ github.workspace }}
        type: string
      RUN_SONAR_SCAN:
        required: false
        default: true
        description: Enable Sonar scan
        type: boolean
      SKIP_SEMANTIC:
        required: false
        type: boolean
        default: false
      VULN_THRESHOLD_SAST_HIGH:
        required: false
        type: number
        default: 10
        description: "Used for checkmarx only"
      VULN_THRESHOLD_SAST_MED:
        required: false
        type: number
        default: 20
        description: "Used for checkmarx only"
      VULN_THRESHOLD_SAST_LOW:
        required: false
        type: number
        default: 200
        description: "Used for checkmarx only"
      VULN_THRESHOLD_SAST:
        required: false
        type: string
        default: "high"
        description: |
          severity to fail pipeline for security vulnerabilities for non-checkmarx SAST scans. 
          If any vulnerability is found at or over the given threshold, then the job will fail. 
          Select “none” to disable.
          Options are: none, low, medium, high
      ENABLE_PDF_REPORT:
        description: Will generate a PDF report from Checkmarx if true
        type: string
        default: ''
      VULN_THRESHOLD_CONTAINER:
        description: |
          Severity to fail pipeline for security vulnerabilities. 
          If any vulnerability is found at or over the given threshold, then the job will fail. 
          Select “none” to disable.
          Options are: none, low, medium, high, critical
        type: string
        default: "critical"
      COMP_THRESHOLD_CONTAINER:
        description: |
          severity to fail pipeline for security vulnerabilities. 
          If any vulnerability is found at or over the given threshold, then the job will fail. 
          Select “none” to disable.
          Options are: none, low, medium, high, critical
        type: string
        default: "high"
      ARTIFACT_SCA:
        required: false
        type: string
        description: Normally this is fine to leave blank, but if Snyk is having a hard time finding it, it can be defined here.
      WORKING_DIRS_SCA:
        description: Comma-separated list of directories to perform SCA scanning on
        default: '.'
        type: string
      VULN_THRESHOLD_SCA:
        description: |
          Severity to fail pipeline for security vulnerabilities. 
          If any vulnerability is found at or over the given threshold, then the job will fail. 
          Select “none” to disable.
          Options are: none, low, medium, high, critical
        type: string
        default: "critical"
      SCA_TOOL:
        description: Which SCA scanning tool to use. Valid options are "xray" and "snyk"
        type: string
        default: snyk
      CONTAINER_SCAN_TOOL:
        description: Which SCA scanning tool to use. Valid options are "prisma" and "snyk"
        type: string
        default: snyk
      SAST_TOOL:
        description: Which SCA scanning tool to use. Valid options are "codeql" and "snyk"
        type: string
        default: snyk
      SNYK_ORG:
        description: id of the Snyk org to monitor project under
        type: string
        required: false
        default: cvs-health-default
      EXTRA_PARAMETERS_SNYK:
        description: Additional parameters to add to scan command
        type: string
        default: ''
      ON_RELEASE_BRANCH:
        description: |
          Is this branch a release branch? By default, it's trunk-based development
          valid availabe options:
          - 'trunk-based'
          - 'true'
          - 'false'
        type: string
        default: 'trunk-based'
      WATCHES_XRAY:
        description: Comma-separated list of Xray watches to filter vulnerabilities. By default, all vulnerabilities will be shown.
        default: 'Global_Vuln_Watch,Global_License_Violation'
        type: string
      ALL_PROJECTS_SNYK:
        required: false
        type: string
        description: scan all dependency files in a repo
      SKIP_UNRESOLVED_SNYK:
        required: false
        type: string
        description: Only use if you are already installing dependencies, and if the Snyk output is recommending this option
      GENERATE_SEMANTIC_RELEASERC_FILE:
        default: true
        type: boolean
      ARTIFACT_NAME:
        type: string
        default: ${{ github.event.repository.name }}
        description: "The name of the artifact to be built. The repository name will be used by default."
      IMAGE_TAG_VERSION:
        type: string
        default: ""
        description: "The image tag version that needs to used instead of semantic version"
      USE_LATEST_TAG:
        description: |
          This tag will be used when building the docker image. If empty, this will be automatically done based on on-release-branch.if its set to true then latest tag would be applied
        type: string
        default: 'false'
      DOCKER_REGISTRY_NAME:
        description: Jfrog artifactory docker registry name defaulted to cvsdigital-docker
        required: false
        type: string
        default: "cvsdigital-docker"
      DOCKERFILE_PATH:
        type: string
        default: "./Dockerfile"
      DOCKER_PUBLISH:
        type: boolean
        default: true
      DOCKER_BUILD_FLAGS:
        description: Additional flags to be used in the docker build command, if required.
        default: ''
        type: string
      LIBRARY_PUBLISH:
        description: Publish Library Artifact
        required: false
        default: false
        type: boolean
      HARNESS_DEPLOY_FILE:
        required: false
        default: "NA"
        type: string
      HARNESS_IMAGE_PR_REFRESH_TIMEOUT:
        description: "Default refresh timeout"
        type: number
        default: 30
      HARNESS_IMAGE_PR_SLEEP_TIMEOUT:
        description: "Default Sleep timeout"
        type: number
        default: 20
      GITHUB_ORG_NAME:
        required: false
        default: "${{ github.repository_owner }}"
        type: string
      GITHUB_REPO_NAME:
        required: false
        default: "${{ github.event.repository.name }}"
        type: string
      SONAR_INPUT_ARGS:
        required: false
        default: "-Dsonar.java.binaries=build/reports"
        type: string
      SONAR_JAVA_VERSION:
        required: false
        type: string
        default: "18"
      SONAR_QUALITY_GATE:
        default: sonar_devex
        description: Quality gate that needs to be applied for sonar scan
        type: string
      CODEQL_SOURCE_DIRECTORY:
        description: (optional mainly for monorepo) source folder for app
        type: string
        default: "none"
      CODEQL_BUILD_COMMAND:
        description: build command for codeql if project is a compiled and using non standard build steps https://docs.github.com/en/code-security/codeql-cli/getting-started-with-the-codeql-cli/preparing-your-code-for-codeql-analysis#creating-databases-for-compiled-languages
        type: string
        default: "none"
      CODEQL_LANGUAGE:
        description: |
          Is this codeql language? By default, it's auto 
          valid availabe options:
          - "c-cpp"
          - "csharp"
          - "java-kotlin"
          - "go"
          - "python"
          - "ruby"
          - "swift"
          - "javascript-typescript"
          - "auto"
        type: string
        default: "auto"
      CODEQL_RAM_LIMIT:
        description: Codeql ram limit for anaylze
        type: number
        default: 3000
      VULN_THRESHOLD_CODEQL_CRITICAL:
        description: Codeql critical threshold for vulnerabilites
        type: number
        default: 10
      VULN_THRESHOLD_CODEQL_HIGH:
        description: Codeql critical threshold for vulnerabilites
        type: number
        default: 20
      VULN_THRESHOLD_CODEQL_MED:
        description: Codeql critical threshold for vulnerabilites
        type: number
        default: 50
      VULN_THRESHOLD_CODEQL_LOW:
        description: Codeql critical threshold for vulnerabilites
        type: number
        default: 100
      SECURITY_GATE_TEAM_NAME:
        description: 'The name of the folder to find the Security Gate Pattern in https://github.com/cvs-health-source-code/team-patterns/security-gate'
        type: string
        default: ''
      FAIL_ON_TESTING:
        description: Block publishing if unit testing fails
        type: boolean
        default: false
        required: false
      HOTFIX_RELEASE:
         required: false
         default: true         
         type: boolean
      HOTFIX_RELEASE_BRANCH:
         required: false
         default: "hotfix"
         type: string
      UPLOAD_SARIF:
         required: false
         default: false
         type: boolean
jobs:
  setup:
    name: "Setup Pipeline"
    runs-on: ${{ inputs.RUNS_ON }}
    outputs:
      ENABLE_GATING: ${{ steps.fetch_pipeline_config.outputs.value }}
    steps:
      - name: Checkout
        uses: actions/checkout@v4
      - name: Set up JDK
        uses: actions/setup-java@v4
        with:
          java-version: ${{ inputs.JAVA_VERSION }}
          distribution: "temurin"
          cache: "gradle"
      - name: Cache Dependencies
        uses: actions/cache/restore@v4
        with:
          path: |
            ~/.gradle/caches
          key: ${{ runner.os }}-${{ hashFiles('**/build.gradle', '**/gradle.properties') }}
      - name: Set required variables
        uses: cvs-health-source-code/gha_workflow_actions/actions/set_env_var@latest
        with:
          SECRETS_CONTEXT: ${{ toJSON(secrets) }}
          VARS_CONTEXT: ${{ toJSON(vars) }}
      - name: Build
        uses: cvs-health-source-code/gha_workflow_actions/actions/gradle_build@latest
        if: ${{ inputs.JAVA_BUILD_COMMAND != 'NA' }}
        with:
          BUILD_COMMAND: ${{ inputs.JAVA_BUILD_COMMAND }}
          ARTIFACTORY_TOKEN: ${{ secrets.JFROG_READER_TOKEN }}
          ARTIFACTORY_USERNAME: ${{ secrets.JFROG_READER_USERNAME }}
      - name: Delete the cache for overwriting
        continue-on-error: true # Don't fail if the cache doesn't exist
        env:
          GH_TOKEN: ${{ github.token }} # required by gh
          CACHE_NAME: ${{ runner.os }}-${{ hashFiles('**/build.gradle', '**/gradle.properties') }}
        run: |
          wget https://github.com/cli/cli/releases/download/v2.63.2/gh_2.63.2_linux_amd64.tar.gz
          # Extract the archive
          tar -xvf gh_2.63.2_linux_amd64.tar.gz
          # Rename
          mkdir -p $HOME/.local/bin/
          cp -f gh_2.63.2_linux_amd64/bin/* $HOME/.local/bin/ >/dev/null
          ls $HOME/.local/bin/
          export PATH=${PATH}:${HOME}/.local/bin/
          gh --version
          gh cache delete "${CACHE_NAME}" || true
      - name: Cache Dependencies
        uses: actions/cache/save@v4
        with:
          path: |
            ~/.gradle/caches
          key: ${{ runner.os }}-${{ hashFiles('**/build.gradle', '**/gradle.properties') }}
      - name: Cache Build Artifacts
        uses: actions/cache/save@v4
        with:
          path: |
            build/*
          key: ${{ runner.os }}-${{ github.run_id }}
      - name: Generate a GH auth token
        id: generate_token
        uses: actions/create-github-app-token@v1
        with:
          app-id: ${{ secrets.GHA_WORKFLOW_APP_ID }}
          private-key: ${{ secrets.GHA_WORKFLOW_APP_PRIVATE_KEY }}
          owner: "cvs-health-source-code"
      - name: Get Pipeline Config
        id: fetch_pipeline_config
        uses: cvs-health-source-code/gha_workflow_actions/actions/get_pipeline_config@latest
        with:
          gh-token: ${{ steps.generate_token.outputs.token }}

  sast-scan:
    name: SAST Scan
    runs-on: ${{ inputs.RUNS_ON }}
    needs: setup
    steps:
      - name: Checkout
        uses: actions/checkout@v4
      - name: restore cache
        uses: actions/cache/restore@v4
        id: cache
        with:
          path: |
            ~/.gradle/caches
          key: ${{ runner.os }}-${{ hashFiles('**/build.gradle', '**/gradle.properties') }}
      - name: Set up JDK
        uses: actions/setup-java@v4
        with:
          java-version: ${{ inputs.JAVA_VERSION }}
          distribution: "temurin"
          cache: "gradle"
      - name: Initiate SAST Scan (Snyk)
        if: env.SAST_TOOL == 'snyk'
        uses: cvs-health-source-code/secure-pipeline-scans/gha/sast-scan/snyk@v2
        with:
          snyk-token: ${{ secrets.SNYK_TOKEN }}
          snyk-org: ${{ inputs.SNYK_ORG }}
          gh-app-private-key: ${{ secrets.GHA_WORKFLOW_APP_PRIVATE_KEY }}
          gh-app-id: ${{ secrets.GHA_WORKFLOW_APP_ID }}
          github-app-key: ${{ secrets.SECOPS_GH_APP_KEY }}
          workspace: ${{ github.workspace }}
          severity-threshold: ${{ inputs.VULN_THRESHOLD_SAST }}
          cache-key: ${{ runner.os }}-${{ github.run_id }}-${{ hashFiles('.github') }}
          dso-kafka-key: ${{ secrets.DSO_KAFKA_API_KEY }}
          dso-kafka-secret: ${{ secrets.DSO_KAFKA_API_SECRET }}
          dso-kafka-prod-key: ${{ secrets.KAFKA_PROD_API_KEY }}
          dso-kafka-prod-secret: ${{ secrets.KAFKA_PROD_API_SECRET }}

  sca-scan:
    name: "SCA Security Scan"
    needs: setup
    runs-on: ${{ inputs.RUNS_ON }}
    steps:
      - name: Checkout
        uses: actions/checkout@v4
      - name: restore cache
        uses: actions/cache/restore@v4
        id: cache
        with:
          path: |
            ~/.gradle/caches
          key: ${{ runner.os }}-${{ hashFiles('**/build.gradle', '**/gradle.properties') }}
      - name: Restore Build Artifacts
        uses: actions/cache/restore@v4
        with:
          path: |
            build/*
          key: ${{ runner.os }}-${{ github.run_id }}
      - name: Set up JDK
        uses: actions/setup-java@v4
        with:
          java-version: ${{ inputs.JAVA_VERSION }}
          distribution: "temurin"
          cache: "gradle"
      - name: Artifactory Setup
        uses: cvs-health-source-code/gha_workflow_actions/actions/gradle_setup_install@latest
        with:
          ARTIFACTORY_USERNAME: ${{ secrets.JFROG_READER_USERNAME }}
          ARTIFACTORY_TOKEN:  ${{ secrets.JFROG_READER_TOKEN }}
      - name: Initiate SCA Scan (Snyk)
        uses: cvs-health-source-code/Secure-Pipeline-Scans/gha/sca-scan/snyk@v2
        if: ${{ env.SCA_TOOL == 'snyk' }}
        with:
          snyk-token: ${{ secrets.SNYK_TOKEN }}
          snyk-org: ${{ inputs.SNYK_ORG }}
          github-app-key: ${{ secrets.SECOPS_GH_APP_KEY }}
          artifactory_username: ${{ secrets.JFROG_READER_USERNAME }}
          artifactory_token: ${{ secrets.JFROG_READER_TOKEN }}
          workspace: ${{ github.workspace }}
          working-dirs: ${{ inputs.WORKING_DIRS_SCA }}
          severity-threshold: ${{ inputs.VULN_THRESHOLD_SCA }}
          extra-parameters: ${{ inputs.EXTRA_PARAMETERS_SNYK }}
          artifact: ${{ inputs.ARTIFACT_SCA }}
          all-projects: ${{ inputs.ALL_PROJECTS_SNYK }}
          skip-unresolved: ${{ inputs.SKIP_UNRESOLVED_SNYK }}
          security_gate_team_name: ${{ inputs.SECURITY_GATE_TEAM_NAME }}
          cache-key: ${{ runner.os }}-${{ github.run_id }}-${{ hashFiles('.github') }}
          dso-kafka-key: ${{ secrets.DSO_KAFKA_API_KEY }}
          dso-kafka-secret: ${{ secrets.DSO_KAFKA_API_SECRET }}
          dso-kafka-prod-key: ${{ secrets.KAFKA_PROD_API_KEY }}
          dso-kafka-prod-secret: ${{ secrets.KAFKA_PROD_API_SECRET }}
          gh-app-id: ${{ secrets.GHA_WORKFLOW_APP_ID }}
          gh-app-private-key: ${{ secrets.GHA_WORKFLOW_APP_PRIVATE_KEY }}

  test:
    name: Test
    runs-on: ${{ inputs.RUNS_ON }}
    if: ${{ inputs.JAVA_TEST_COMMAND != 'NA' }}
    needs: setup
    steps:
      - name: Checkout
        uses: actions/checkout@v4
      - name: restore cache
        uses: actions/cache/restore@v4
        id: cache
        with:
          path: |
            ~/.gradle/caches
          key: ${{ runner.os }}-${{ hashFiles('**/build.gradle', '**/gradle.properties') }}
      - name: Restore Build Artifacts
        uses: actions/cache/restore@v4
        with:
          path: |
            build/*
          key: ${{ runner.os }}-${{ github.run_id }}
      - name: Set up JDK
        uses: actions/setup-java@v4
        with:
          java-version: ${{ inputs.JAVA_VERSION }}
          distribution: "temurin"
          cache: "gradle"
      - name: Set required variables
        uses: cvs-health-source-code/gha_workflow_actions/actions/set_env_var@latest
        with:
          SECRETS_CONTEXT: ${{ toJSON(secrets) }}
          VARS_CONTEXT: ${{ toJSON(vars) }}
      - name: Test
        uses: cvs-health-source-code/gha_workflow_actions/actions/gradle_test@latest
        with:
          ARTIFACTORY_TOKEN: ${{ secrets.JFROG_READER_TOKEN }}
          ARTIFACTORY_USERNAME: ${{ secrets.JFROG_READER_USERNAME }}
          TEST_COMMAND: ${{ inputs.JAVA_TEST_COMMAND }}
      - name: Generate a GH auth token
        id: generate_token
        uses: cvs-health-source-code/cvs-gha-github-app-token@v2.1.0
        with:
          app_id: ${{ secrets.GHA_WORKFLOW_APP_ID }}
          private_key: ${{ secrets.GHA_WORKFLOW_APP_PRIVATE_KEY }}
      - name: Run SonarQube
        if: ${{ inputs.RUN_SONAR_SCAN != false }}
        uses: cvs-health-source-code/gha_workflow_actions/actions/sonar_scan@latest
        with:
          SONAR_TOKEN: ${{ secrets.SONAR_TOKEN }}
          PROJECT_NAME: ${{ inputs.GITHUB_ORG_NAME }}_${{ inputs.GITHUB_REPO_NAME }}
          SONAR_JAVA_VERSION: ${{ inputs.SONAR_JAVA_VERSION }}
          GITHUB_ORG: ${{ inputs.GITHUB_ORG_NAME }}
          PROJECT_KEY: "${{ inputs.GITHUB_ORG_NAME }}_${{ inputs.GITHUB_REPO_NAME }}"
          INPUT_ARGS: ${{ inputs.SONAR_INPUT_ARGS }}
          SONAR_QUALITY_GATE: ${{ inputs.SONAR_QUALITY_GATE }}
          SECURITY_GATE_TEAM_NAME: ${{ inputs.SECURITY_GATE_TEAM_NAME }}
          GITHUB_TOKEN: ${{ steps.generate_token.outputs.token }}


  lint:
    name: "Lint"
    needs: setup
    runs-on: ${{ inputs.RUNS_ON }}
    if: ${{ inputs.JAVA_LINT_COMMAND != 'NA' }}
    steps:
      - name: Checkout
        uses: actions/checkout@v4
      - name: restore cache
        uses: actions/cache/restore@v4
        id: cache
        with:
          path: |
            ~/.gradle/caches
          key: ${{ runner.os }}-${{ hashFiles('**/build.gradle', '**/gradle.properties') }}
      - name: Restore Build Artifacts
        uses: actions/cache/restore@v4
        with:
          path: |
            build/*
          key: ${{ runner.os }}-${{ github.run_id }}
      - name: Set up JDK
        uses: actions/setup-java@v4
        with:
          java-version: ${{ inputs.JAVA_VERSION }}
          distribution: "temurin"
          cache: "gradle"
      - name: Lint
        uses: cvs-health-source-code/gha_workflow_actions/actions/gradle_lint@latest
        with:
          LINT_COMMAND: ${{ inputs.JAVA_LINT_COMMAND }}

  release-version:
    name: Release Version
    runs-on: ${{ inputs.RUNS_ON }}
    outputs:
      release_tag: ${{ steps.release_tag.outputs.RELEASE_TAG }}
      branch_name: ${{ steps.extract_branch.outputs.GHA_BRANCH }}
    env:
      GIT_COMMIT_SHA: ${{ github.sha }}
    needs: setup
    if: ${{  toJSON(inputs.SKIP_SEMANTIC) == 'false' }} || ${{ inputs.IMAGE_TAG_VERSION != '' }}
    steps:
      - name: Checkout code
        uses: actions/checkout@v4
      - name: Custom Image tag
        if: ${{ inputs.IMAGE_TAG_VERSION != '' }}
        run: |
          echo "Utilizing custom image tag: ${{ inputs.IMAGE_TAG_VERSION }}"
          echo "${{  inputs.IMAGE_TAG_VERSION }}"
          rm -f release.version
          echo "${{ inputs.IMAGE_TAG_VERSION }}" > release.version
          cat release.version
      - name: Semantic Release
        if: ${{ inputs.IMAGE_TAG_VERSION == '' && toJSON(inputs.SKIP_SEMANTIC) == 'false' }}
        uses: cvs-health-source-code/gha_workflow_actions/actions/semantic_release@latest
        with:
          WITH_RELEASERC:  ${{ toJSON(inputs.GENERATE_SEMANTIC_RELEASERC_FILE) }}
          APP_ID: ${{ secrets.GHA_WORKFLOW_APP_ID }}
          PRIVATE_KEY: ${{ secrets.GHA_WORKFLOW_APP_PRIVATE_KEY }}
      - name: Assign release tag
        id: release_tag
        run: |
          if [ ! -f release.version ]
          then
            echo "Release version not exists as there were no release"
            touch release.version
          fi
          TAG_VERSION=$(cat release.version)
          if [[ -z "$TAG_VERSION" ]]; then
            echo "TAG_VERSION was not found into release.version file using GIT_COMMIT_SHA value ${GIT_COMMIT_SHA} as TAG_VERSION"
            TAG_VERSION="${GIT_COMMIT_SHA}"
          fi
          echo "RELEASE_TAG=$TAG_VERSION" >> "$GITHUB_OUTPUT"
      - name: Branch used
        id: extract_branch
        run: |
          if [[ "${GITHUB_EVENT_NAME}" == "push" ]]; then
            echo "GHA_BRANCH=$(echo ${GITHUB_REF##*/})" >> $GITHUB_OUTPUT
          elif [[ "${GITHUB_EVENT_NAME}" == "pull_request" ]]; then
            echo "GHA_BRANCH=$(echo $GITHUB_BASE_REF)" >> $GITHUB_OUTPUT
          else
            echo "No branch"
            echo "GHA_BRANCH=$(echo $DUMMY)" >> $GITHUB_OUTPUT
          fi

  push-artifact:
    name: Push Artifact
    runs-on: ${{ inputs.RUNS_ON }}
    needs: [release-version, setup]
    if: | 
      inputs.LIBRARY_PUBLISH &&
      always() &&
      !contains(needs.*.result, 'failure') &&
      !contains(needs.*.result, 'cancelled') &&
      needs.setup.outputs.ENABLE_GATING != 'true'
    steps:
      - name: Checkout
        uses: actions/checkout@v4
      - name: restore cache
        uses: actions/cache/restore@v4
        id: cache
        with:
          path: |
            ~/.gradle/caches
          key: ${{ runner.os }}-${{ hashFiles('**/build.gradle', '**/gradle.properties') }}
      - name: Restore Build Artifacts
        uses: actions/cache/restore@v4
        with:
          path: |
            build/*
          key: ${{ runner.os }}-${{ github.run_id }}
      - name: Set up JDK
        uses: actions/setup-java@v4
        with:
          java-version: ${{ inputs.JAVA_VERSION }}
          distribution: "temurin"
          cache: "gradle"
      - name: Setup Jfrog Auth
        uses: cvs-health-source-code/gha_workflow_actions/actions/gradle_setup_install@latest
        with:
          ARTIFACTORY_TOKEN: ${{ secrets.ARTIFACTORY_TOKEN }}
          ARTIFACTORY_USERNAME: ${{ secrets.ARTIFACTORY_USERNAME }}
      - name: Publish JAR
        uses: cvs-health-source-code/gha_workflow_actions/actions/gradle_publish@latest
        with:
          ARTIFACTORY_TOKEN: ${{ secrets.ARTIFACTORY_TOKEN }}
          ARTIFACTORY_USERNAME: ${{ secrets.ARTIFACTORY_USERNAME }}
          PUBLISH_COMMAND: ${{ inputs.JAVA_PUBLISH_COMMAND }}

  push-artifact-gated:
    name: Push Artifact (Gated)
    runs-on: ${{ inputs.RUNS_ON }}
    needs: [ release-version, setup, sast-scan, sca-scan, container-scan ]
    if: |
      inputs.LIBRARY_PUBLISH &&
      always() &&
      !contains(needs.*.result, 'failure') &&
      !contains(needs.*.result, 'cancelled') && 
      contains(needs.sca-scan.result, 'success') &&
      contains(needs.sast-scan.result, 'success') &&
      contains(needs.container-scan.result, 'success') &&
      needs.setup.outputs.ENABLE_GATING == 'true'
    steps:
      - name: Checkout
        uses: actions/checkout@v4
      - name: restore cache
        uses: actions/cache/restore@v4
        id: cache
        with:
          path: |
            ~/.gradle/caches
          key: ${{ runner.os }}-${{ hashFiles('**/build.gradle', '**/gradle.properties') }}
      - name: Restore Build Artifacts
        uses: actions/cache/restore@v4
        with:
          path: |
            build/*
          key: ${{ runner.os }}-${{ github.run_id }}
      - name: Set up JDK
        uses: actions/setup-java@v4
        with:
          java-version: ${{ inputs.JAVA_VERSION }}
          distribution: "temurin"
          cache: "gradle"
      - name: Setup Jfrog Auth
        uses: cvs-health-source-code/gha_workflow_actions/actions/gradle_setup_install@latest
        with:
          ARTIFACTORY_TOKEN: ${{ secrets.ARTIFACTORY_TOKEN }}
          ARTIFACTORY_USERNAME: ${{ secrets.ARTIFACTORY_USERNAME }}
      - name: Publish JAR
        uses: cvs-health-source-code/gha_workflow_actions/actions/gradle_publish@latest
        with:
          ARTIFACTORY_TOKEN: ${{ secrets.ARTIFACTORY_TOKEN }}
          ARTIFACTORY_USERNAME: ${{ secrets.ARTIFACTORY_USERNAME }}
          PUBLISH_COMMAND: ${{ inputs.JAVA_PUBLISH_COMMAND }}

  docker-build:
    name: "Docker Build"
    runs-on: ${{ inputs.RUNS_ON }}
    needs: [release-version]
    env:
      RELEASE_TAG: ${{needs.release-version.outputs.release_tag}}
      CIRCLE_BRANCH: ${{needs.release-version.outputs.branch_name}}
      CIRCLE_PROJECT_REPONAME: ${{ github.event.repository.name }}
    if: | 
      always() &&
      !contains(needs.*.result, 'failure') &&
      !contains(needs.*.result, 'cancelled')
    steps:
      - name: Checkout
        uses: actions/checkout@v4
      - name: restore cache
        uses: actions/cache/restore@v4
        id: cache
        with:
          path: |
            ~/.gradle/caches
          key: ${{ runner.os }}-${{ hashFiles('**/build.gradle', '**/gradle.properties') }}
      - name: Restore Build Artifacts
        uses: actions/cache/restore@v4
        with:
          path: |
            build/*
          key: ${{ runner.os }}-${{ github.run_id }}
      - name: Cache Semantic Release
        uses: actions/cache@v4
        with:
          path: release.version
          key: ${{ runner.os }}-${{ hashFiles('release.version') }}
      - name: "Determine if release branch"
        id: "release_branch"
        run: |
           branch=${GITHUB_HEAD_REF:-${GITHUB_REF#refs/heads/}}
           echo "Detected branch: $branch"
           if [[ "${{ inputs.SKIP_SEMANTIC }}" == "false" && \
               ($branch == "main" || $branch == "master" || \
               (${{ inputs.HOTFIX_RELEASE }} == true && $branch == ${{ inputs.HOTFIX_RELEASE_BRANCH }} )) ]]; then
               echo "on_release_branch='true'"
               branch=${{ inputs.HOTFIX_RELEASE_BRANCH }}
               echo "Detected branch: $branch"
           else 
             echo "on_release_branch=''"
             branch=${GITHUB_HEAD_REF:-${GITHUB_REF#refs/heads/}}
             echo "Detected branch: $branch"
           fi
      - name: Set up JDK
        uses: actions/setup-java@v4
        with:
          java-version: ${{ inputs.JAVA_VERSION }}
          distribution: "temurin"
          cache: "gradle"
      - name: Release setup
        run:
          echo "$RELEASE_TAG" > release.version
      - name: Docker Build
        uses: cvs-health-source-code/secure-pipeline-scans/gha/docker-build@v2
        with:
          dockerfile: ${{ inputs.DOCKERFILE_PATH }}
          artifact_name: ${{ inputs.ARTIFACT_NAME }}
          docker_registry_name: ${{ inputs.DOCKER_REGISTRY_NAME }}
          release-tag: ${{ env.RELEASE_TAG }}
          artifactory_token: ${{ secrets.JFROG_READER_TOKEN }}
          artifactory_username: ${{ secrets.JFROG_READER_USERNAME }}
          gh-app-id: ${{ secrets.GHA_WORKFLOW_APP_ID }}
          gh-app-private-key: ${{ secrets.GHA_WORKFLOW_APP_PRIVATE_KEY }}
          rally-api-key: ${{ secrets.RALLY_API_KEY }}
          docker-build-flags: ${{ inputs.DOCKER_BUILD_FLAGS }}
          cache-key: ${{ runner.os }}-${{ github.run_id }}-${{ hashFiles('.github') }}

  container-scan:
    name: "Container Scan"
    runs-on: ${{ inputs.RUNS_ON }}
    needs: [setup, docker-build]
    if: | 
      always() &&
      !contains(needs.*.result, 'failure') &&
      !contains(needs.*.result, 'cancelled')
    steps:
      - name: Checkout
        uses: actions/checkout@v4
      - name: restore cache
        uses: actions/cache/restore@v4
        id: cache
        with:
          path: |
            ~/.gradle/caches
          key: ${{ runner.os }}-${{ hashFiles('**/build.gradle', '**/gradle.properties') }}
      - name: Restore Build Artifacts
        uses: actions/cache/restore@v4
        with:
          path: |
            build/*
          key: ${{ runner.os }}-${{ github.run_id }}
      - name: Set up JDK
        uses: actions/setup-java@v4
        with:
          java-version: ${{ inputs.JAVA_VERSION }}
          distribution: "temurin"
          cache: "gradle"
      - name: Initiate Container Scan (Prisma)
        uses: cvs-health-source-code/secure-pipeline-scans/gha/container-scan@v2
        if: ${{ env.CONTAINER_SCAN_TOOL != 'snyk' }}
        with:
          tl_console_url: ${{ secrets.TL_CONSOLE_URL }}
          tl_user: ${{ secrets.TL_USER }}
          tl_pass: ${{ secrets.TL_PASS }}
          artifactory_token: ${{ secrets.JFROG_READER_TOKEN }}
          artifactory_username: ${{ secrets.JFROG_READER_USERNAME }}
          github-app-key: ${{ secrets.SECOPS_GH_APP_KEY }}
          vuln_threshold: ${{ inputs.VULN_THRESHOLD_CONTAINER }}
          comp_threshold: ${{ inputs.COMP_THRESHOLD_CONTAINER }}
          dso-kafka-key: ${{ secrets.DSO_KAFKA_API_KEY }}
          dso-kafka-secret: ${{ secrets.DSO_KAFKA_API_SECRET }}
          dso-kafka-prod-key: ${{ secrets.KAFKA_PROD_API_KEY }}
          dso-kafka-prod-secret: ${{ secrets.KAFKA_PROD_API_SECRET }}
          gh-app-id: ${{ secrets.GHA_WORKFLOW_APP_ID }}
          gh-app-private-key: ${{ secrets.GHA_WORKFLOW_APP_PRIVATE_KEY }}
          security_gate_team_name: ${{ inputs.SECURITY_GATE_TEAM_NAME }}
          cache-key: ${{ runner.os }}-${{ github.run_id }}-${{ hashFiles('.github') }}
      - name: Initiate Container Scan (Snyk)
        uses: cvs-health-source-code/secure-pipeline-scans/gha/container-scan/snyk@v2
        if: ${{ env.CONTAINER_SCAN_TOOL == 'snyk' }}
        with:
          snyk-token: ${{ secrets.SNYK_TOKEN }}
          snyk-org: ${{ inputs.SNYK_ORG }}
          artifactory_token: ${{ secrets.JFROG_READER_TOKEN }}
          artifactory_username: ${{ secrets.JFROG_READER_USERNAME }}
          github-app-key: ${{ secrets.SECOPS_GH_APP_KEY }}
          vuln_threshold: ${{ inputs.VULN_THRESHOLD_CONTAINER }}
          dso-kafka-key: ${{ secrets.DSO_KAFKA_API_KEY }}
          dso-kafka-secret: ${{ secrets.DSO_KAFKA_API_SECRET }}
          dso-kafka-prod-key: ${{ secrets.KAFKA_PROD_API_KEY }}
          dso-kafka-prod-secret: ${{ secrets.KAFKA_PROD_API_SECRET }}
          gh-app-id: ${{ secrets.GHA_WORKFLOW_APP_ID }}
          gh-app-private-key: ${{ secrets.GHA_WORKFLOW_APP_PRIVATE_KEY }}
          security_gate_team_name: ${{ inputs.SECURITY_GATE_TEAM_NAME }}
          cache-key: ${{ runner.os }}-${{ github.run_id }}-${{ hashFiles('.github') }}

  docker-publish:
    name: "Docker Publish"
    runs-on: ${{ inputs.RUNS_ON }}
    needs: [setup, docker-build, test]
    if: | 
      always() &&
      !contains(needs.setup.result, 'failure') &&
      !contains(needs.setup.result, 'cancelled') &&
      !contains(needs.docker-build.result, 'failure') &&
      !contains(needs.docker-build.result, 'cancelled') &&
      !contains(needs.docker-build.result, 'skipped') &&
      inputs.DOCKER_PUBLISH == true &&
      !(inputs.FAIL_ON_TESTING == true && needs.test.result == 'failure') &&
      needs.setup.outputs.ENABLE_GATING != 'true'
    steps:
      - name: Checkout
        uses: actions/checkout@v4
      - name: restore cache
        uses: actions/cache/restore@v4
        id: cache
        with:
          path: |
            ~/.gradle/caches
          key: ${{ runner.os }}-${{ hashFiles('**/build.gradle', '**/gradle.properties') }}
      - name: Restore Build Artifacts
        uses: actions/cache/restore@v4
        with:
          path: |
            build/*
          key: ${{ runner.os }}-${{ github.run_id }}
      - name: Set up JDK
        uses: actions/setup-java@v4
        with:
          java-version: ${{ inputs.JAVA_VERSION }}
          distribution: "temurin"
          cache: "gradle"
      - name: Docker Publish
        uses: cvs-health-source-code/secure-pipeline-scans/gha/docker-push@v2
        with:
          artifactory_token: ${{ secrets.ARTIFACTORY_TOKEN }}
          artifactory_username: ${{ secrets.ARTIFACTORY_USERNAME }}
          use-latest-tag: ${{ inputs.USE_LATEST_TAG }}
          cache-key: ${{ runner.os }}-${{ github.run_id }}-${{ hashFiles('.github') }}

  docker-publish-gated:
    name: "Docker Publish (Gated)"
    runs-on: ${{ inputs.RUNS_ON }}
    needs: [ setup, docker-build, test, sca-scan, sast-scan, container-scan ]
    if: |
      always() &&
      !contains(needs.setup.result, 'failure') &&
      !contains(needs.setup.result, 'cancelled') &&
      !contains(needs.docker-build.result, 'failure') &&
      !contains(needs.docker-build.result, 'cancelled') &&
      !contains(needs.docker-build.result, 'skipped') &&
      contains(needs.sca-scan.result, 'success') &&
      contains(needs.sast-scan.result, 'success') &&
      contains(needs.container-scan.result, 'success') &&
      inputs.DOCKER_PUBLISH == true &&
      !(inputs.FAIL_ON_TESTING == true && needs.test.result == 'failure') &&
      needs.setup.outputs.ENABLE_GATING == 'true'
    steps:
      - name: Checkout
        uses: actions/checkout@v4
      - name: restore cache
        uses: actions/cache/restore@v4
        id: cache
        with:
          path: |
            ~/.gradle/caches
          key: ${{ runner.os }}-${{ hashFiles('**/build.gradle', '**/gradle.properties') }}
      - name: Restore Build Artifacts
        uses: actions/cache/restore@v4
        with:
          path: |
            build/*
          key: ${{ runner.os }}-${{ github.run_id }}
      - name: Set up JDK
        uses: actions/setup-java@v4
        with:
          java-version: ${{ inputs.JAVA_VERSION }}
          distribution: "temurin"
          cache: "gradle"
      - name: Docker Publish
        uses: cvs-health-source-code/secure-pipeline-scans/gha/docker-push@v2
        with:
          artifactory_token: ${{ secrets.ARTIFACTORY_TOKEN }}
          artifactory_username: ${{ secrets.ARTIFACTORY_USERNAME }}
          use-latest-tag: ${{ inputs.USE_LATEST_TAG }}
          cache-key: ${{ runner.os }}-${{ github.run_id }}-${{ hashFiles('.github') }}

  sbom:
    name: "Generate SBOM"
    runs-on: ${{ inputs.RUNS_ON }}
    needs: [sca-scan, container-scan, sast-scan, docker-publish, docker-publish-gated]
    if: | 
      always() &&
      (!contains(needs.docker-publish.result, 'failure') &&
      !contains(needs.docker-publish.result, 'skipped') &&
      !contains(needs.docker-publish.result, 'cancelled')) ||
      (!contains(needs.docker-publish-gated.result, 'failure') &&
      !contains(needs.docker-publish-gated.result, 'skipped') &&
      !contains(needs.docker-publish-gated.result, 'cancelled'))
    steps:
      - name: Checkout
        uses: actions/checkout@v4
      - name: Generate SBOM
        uses: cvs-health-source-code/secure-pipeline-scans/gha/attest@v2
        with:
          artifactory_token: ${{ secrets.ARTIFACTORY_DEVSECOPS_ATTEST_TOKEN }}
          cosign_key: ${{ secrets.COSIGN_KEY }}
          dso-kafka-key: ${{ secrets.DSO_KAFKA_API_KEY }}
          dso-kafka-secret: ${{ secrets.DSO_KAFKA_API_SECRET }}
          dso-kafka-prod-key: ${{ secrets.KAFKA_PROD_API_KEY }}
          dso-kafka-prod-secret: ${{ secrets.KAFKA_PROD_API_SECRET }}
          cache-key: ${{ runner.os }}-${{ github.run_id }}-${{ hashFiles('.github') }}
          tool: "all"

  harness-values-update:
    name: "Update Harness Values"
    runs-on: ${{ inputs.RUNS_ON }}
    needs: [setup, release-version, docker-publish, docker-publish-gated]
    env:
      RELEASE_TAG: ${{needs.release-version.outputs.release_tag}}
    if: | 
      always() &&
      !contains(needs.setup.result, 'failure') &&
      !contains(needs.setup.result, 'skipped') &&
      !contains(needs.setup.result, 'cancelled') &&
      ((!contains(needs.docker-publish.result, 'failure') &&
      !contains(needs.docker-publish.result, 'skipped') &&
      !contains(needs.docker-publish.result, 'cancelled')) ||
      (!contains(needs.docker-publish-gated.result, 'failure') &&
      !contains(needs.docker-publish-gated.result, 'skipped') &&
      !contains(needs.docker-publish-gated.result, 'cancelled'))) &&
      inputs.HARNESS_DEPLOY_FILE != 'NA'
    steps:
      - name: Checkout
        uses: actions/checkout@v4
      - name: Cache Semantic Release
        uses: actions/cache@v4
        with:
          path: release.version
          key: ${{ runner.os }}-${{ hashFiles('release.version') }}
      - name: Release setup
        run:
          echo $RELEASE_TAG > release.version
      - name: "Update Harness Values"
        uses: cvs-health-source-code/gha_workflow_actions/actions/update_deployment_config@latest
        with:
          build_type: "gradle"
          HARNESS_DEPLOY_FILE: ${{ inputs.HARNESS_DEPLOY_FILE }}
          HARNESS_IMAGE_PR_REFRESH_TIMEOUT: ${{ inputs.HARNESS_IMAGE_PR_REFRESH_TIMEOUT }}
          HARNESS_IMAGE_PR_SLEEP_TIMEOUT: ${{ inputs.HARNESS_IMAGE_PR_SLEEP_TIMEOUT }}
          SKIP_SEMANTIC: ${{ toJSON(inputs.SKIP_SEMANTIC) }}
          IMAGE_TAG_VERSION: ${{ inputs.IMAGE_TAG_VERSION }}
          APP_ID: ${{ secrets.GHA_WORKFLOW_APP_ID }}
          PRIVATE_KEY: ${{ secrets.GHA_WORKFLOW_APP_PRIVATE_KEY }}
          HOTFIX_RELEASE_BRANCH: ${{ inputs.HOTFIX_RELEASE_BRANCH }}
          HOTFIX_RELEASE: ${{ inputs.HOTFIX_RELEASE }}
