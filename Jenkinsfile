pipeline {
    agent any

    // Jenkins cron runs in UTC
    // 9:00 AM IST  = 03:30 UTC
    // 9:00 PM IST  = 15:30 UTC
    triggers {
        cron('''
            30 3 * * *
            30 15 * * *
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

                    # Get IST Time
                    current_time=$(TZ=Asia/Kolkata date +%H:%M)
                    echo "Current IST Time: $current_time"

                    hour=${current_time%:*}
                    minute=${current_time#*:}

                    hour=$((10#$hour))
                    minute=$((10#$minute))

                    total_minutes=$((hour * 60 + minute))

                    # 09:00 AM = 540 mins
                    # 09:00 PM = 1260 mins
                    if [[ "$total_minutes" -ge 540 && "$total_minutes" -lt 1260 ]]; then
                      action="start"
                    else
                      action="stop"
                    fi

                    echo "Final Action: $action"

                    echo "Checking Current Instance State..."
                    current_status=$(aws ec2 describe-instances --instance-ids $INSTANCE_ID --region $REGION --query "Reservations[].Instances[].State.Name" --output text)
                    echo "Current Instance State: $current_status"

                    if [[ "$action" == "start" ]]; then
                      if [[ "$current_status" == "running" ]]; then
                        echo "Instance already running. No action needed."
                      else
                        echo "Starting EC2 Instance..."
                        aws ec2 start-instances --instance-ids $INSTANCE_ID --region $REGION
                      fi
                    fi

                    if [[ "$action" == "stop" ]]; then
                      if [[ "$current_status" == "stopped" ]]; then
                        echo "Instance already stopped. No action needed."
                      else
                        echo "Stopping EC2 Instance..."
                        aws ec2 stop-instances --instance-ids $INSTANCE_ID --region $REGION
                      fi
                    fi

                    echo "Job Completed Successfully"
                    '''
                }
            }
        }
    }
}
