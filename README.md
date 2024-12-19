stages:
  - check-mr-label
  - build
  - dockerbuild

check_mr_label:
  stage: check-mr-label
  script:
    - |
      if [[ -n "$CI_MERGE_REQUEST_ID" ]]; then
      echo "pipeline triggred by a merge request."

      MR_LABELS=$(curl --header "PRIVATE-TOKEN: $CI_JOB_TOKEN" ) \
      "$CI_API_V4_URL/projects/$CI_PROJECT_ID/merge_requests/$CI_MERGE_REQUEST_ID" \
      | jr -r '.labels[]')

      if echo "$MR_LABELS" | grep -q "dev"; then
        echo "merge request has the dev label."
      else 
        echo "merge request does not have the dev label"
      exit 1
      fi
  only: 
    - merge_requests

build:
  stage: build
  image: 
   name:  registry.dev.sbiepay.sbi:8443/ubi9/gradle-8.9-jdk-21:v3
   pull_policy: always
  script:
    - gradle build
    - ls -ltrh build/libs/ | grep -v "plain"
  artifacts:
    paths:
      - build/libs/
    when: on_success 
    expire_in: 1 week
  rules:
    - if: '$CI_PIPELINE_SOURCE == "merge_request_event" && $CI_MERGE_REQUEST_TARGET_BRANCH_NAME == "develop"'
      when: always
    - when: never
  
