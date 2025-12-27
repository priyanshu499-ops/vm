pipeline {
    agent any

    triggers {
        cron('''
            6 16 * * *
            10 16 * * *
        ''')
    }

    stages {

        stage("Test Pipeline Running") {
            steps {
                sh 'echo "Pipeline Successfully Triggered..."'
            }
        }

        stage('Manage AWS EC2 Based on IST Time') {
            steps {
                withAWS(credentials: 'aws-account-tripare', region: 'ap-south-1') {

                    sh '''#!/bin/bash

                    REGION="ap-south-1"
                    INSTANCE_ID="i-02b8041d7c0ccf9ae"

                    echo "=============================="
                    echo " AWS EC2 Auto Start/Stop JOB "
                    echo "=============================="

                    current_time=$(TZ=Asia/Kolkata date +%H:%M)
                    echo "Current IST Time: $current_time"

                    # ------------ TEST LOGIC ------------
                    if [[ "$current_time" == "21:36" ]]; then
                      action="stop"
                    elif [[ "$current_time" == "21:40" ]]; then
                      action="start"
                    else
                      echo "Fallback Default Logic Apply..."
                      hour=${current_time%:*}
                      minute=${current_time#*:}
                      hour=$((10#$hour))
                      minute=$((10#$minute))
                      total_minutes=$((hour * 60 + minute))

                      if [[ "$total_minutes" -ge 540 && "$total_minutes" -lt 1260 ]]; then
                        action="start"
                      else
                        action="stop"
                      fi
                    fi
                    # ------------------------------------

                    echo "Final Action: $action"

                    current_status=$(aws ec2 describe-instances --instance-ids $INSTANCE_ID --region $REGION --query "Reservations[].Instances[].State.Name" --output text)
                    echo "Current Instance State: $current_status"

                    if [[ "$action" == "start" && "$current_status" != "running" ]]; then
                      echo "Starting EC2..."
                      aws ec2 start-instances --instance-ids $INSTANCE_ID --region $REGION
                    fi

                    if [[ "$action" == "stop" && "$current_status" != "stopped" ]]; then
                      echo "Stopping EC2..."
                      aws ec2 stop-instances --instance-ids $INSTANCE_ID --region $REGION
                    fi

                    echo "Job Completed Successfully"
                    '''
                }
            }
        }
    }
}
