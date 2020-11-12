ASBRL-CODESUITE
=========

Create a CloudFormation Template to Deploy a CodeSuite Stack

Requirements
------------

- None


Role Variables
--------------

- TEMPLATE_DEST: (String)
- TAGS:
    - RELEASE: (String)
    - ENVIRONMENT_TYPE: (String)
- PARAMETERS: (List)
  - NAME: (String)
  - TYPE: (String)
- ECR_REPOSITORY:
  - NAME: (String)
- CODEBUILD:
  - NAME: String
  - DEPENDS_ON: (List)
    - (String)
  - DESCRIPTION: (String)
  - SERVICE_ROLE: (String)
  - ARTIFACTS:
    - TYPE: (String)
  - ENVIRONMENT:
    - TYPE: (String)
    - COMPUTE_TYPE: (String)
    - IMAGE: (String)
    - PRIVILEGED_MODE: (Boolean)
    - VARIABLES:
      - NAME: (String)
      - VALUE: (String)
    - SOURCE:
      - TYPE: (String)
    - TIMEOUT_MINUTES: (String)
- CODEPIPELINE: (List)
  - NAME: (String)
  - DEPENDS_ON: (List)
    - (String)
  - RESTART_EXECUTION_ON_UPDATE: (Boolean)
  - ROLE: (String)
  - STAGES: (List)
    - NAME: (String)
    - ACTIONS: (List)
      - NAME: (String)
      - RUN_ORDER: (Number)
      - ACTION_TYPE_ID: 
        - CATEGORY: (String)
        - OWNER: (String)
        - VERSION: (Number)
        - PROVIDER: (String)
      - CONFIGURATION: (List)
        - KEY: (String)
        - VALUE: (String)
      - OUTPUT_ARTIFACTS: (Optional)
        - NAME: (String)
      - INPUT_ARTIFACTS: (Optional)
        - NAME: (String)
- WAIT_CONDITION: (List)
  - NAME: (String)
  - DEPENDS_ON: (String)
  - TIMEOUT: (String)

Dependencies
------------

None

Example Playbook
----------------
      - name: Generate {{EnvironmentType}} cf-codesuite
        include_role:
          name: asbrl-codesuite
        vars:
          TEMPLATE_DEST: ./artifacts/{{EnvironmentType}}/cf-elasticstack-codesuite.yml
          TAGS:
            RELEASE: "{{Release}}"
            ENVIRONMENT_TYPE: "{{EnvironmentType}}"
          PARAMETERS:
          - NAME: CodePipeline
            TYPE: String
          - NAME: CodePipelineKey
            TYPE: String
          - NAME: Build
            TYPE: String
          - NAME: CodebuildTimeout
            TYPE: Number
          - NAME: WaitConditionTimeout
            TYPE: Number
          ECR_REPOSITORY:
          - NAME: Elasticsearch
          CODEBUILD:
          - NAME: ESImage{{ReleaseName}}
            DEPENDS_ON:
            - ECRElasticsearch
            DESCRIPTION: Build Elasticsearch image
            SERVICE_ROLE: CodeBuildES
            ARTIFACTS:
              TYPE: CODEPIPELINE
            ENVIRONMENT:
              TYPE: LINUX_CONTAINER
              COMPUTE_TYPE: BUILD_GENERAL1_SMALL
              IMAGE: aws/codebuild/standard:4.0
              PRIVILEGED_MODE: true
              VARIABLES:
              - NAME: AWS_DEFAULT_REGION
                VALUE: '!Ref AWS::Region'
              - NAME: AWS_ACCOUNT_ID
                VALUE: '!Ref AWS::AccountId'
              - NAME: IMAGE_REPO_NAME
                VALUE: '!Ref ECRElasticsearch'
              - NAME: IMAGE_TAG
                VALUE: '!Ref Build'
              - NAME: WAIT_HANDLE
                VALUE: '!Ref WaitHandleUntilCodeBuildIsReady'
            SOURCE:
              TYPE: CODEPIPELINE
            TIMEOUT_MINUTES: '!Ref CodebuildTimeout'
          CODEPIPELINE:
          - NAME: ESImage
            DEPENDS_ON:
            - CodeBuildESImage{{ReleaseName}}
            RESTART_EXECUTION_ON_UPDATE: true
            ROLE: CodePipelineES
            STAGES: 
            - NAME: Source
            ACTIONS:
            - NAME: SourceAction
              RUN_ORDER: 1
              ACTION_TYPE_ID:
              CATEGORY: Source
              OWNER: AWS
              VERSION: 1
              PROVIDER: S3
              CONFIGURATION:
              - KEY: S3_BUCKET
                VALUE: '!Ref CodePipeline'
              - KEY: S3_KEY
                VALUE: '!Ref CodePipelineKey'
              - KEY: POLL_SOURCE_CHANGES
                VALUE: 'true'
              OUTPUT_ARTIFACTS:
                - NAME: SourceOutput 
            - NAME: Build
              RUN_ORDER: 1
              ACTION_TYPE_ID:
              CATEGORY: Build
              OWNER: AWS
              VERSION: 1
              PROVIDER: CodeBuild
              CONFIGURATION:
              - KEY: ProjectName
                VALUE: '!Ref CodeBuildESImage{{ReleaseName}}'
              INPUT_ARTIFACTS:
              - NAME: SourceOutput
          WAIT_CONDITION:
          - NAME: CodeBuildIsReady
            DEPENDS_ON: CodePipelineESImage
            TIMEOUT: '!Ref WaitConditionTimeout'  
            
License
-------

BSD

Author Information
------------------

Moegui.com
