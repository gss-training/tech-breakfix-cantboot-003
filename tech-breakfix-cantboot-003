#!/bin/bash
MYHOSTNAME=`hostname`
MYNAME=`whoami`

if [[ ! $MYNAME == "root" ]]
then
        echo "$0 needs to be run by the root user."
        exit 3
fi

# startup
logger -p local0.notice "Initiating $0 with option $@"
echo "Initiating $0 with option $@"


break() {
        # resize the parition

        yes | lvreduce -L 500M /dev/mapper/application-app
        echo '/usr/bin/cp /etc/fstab /tmp/fstab-capture.out' >> /etc/rc.d/rc.local
        echo '/usr/bin/cp /proc/self/mounts /tmp/mounts-capture.out' >> /etc/rc.d/rc.local

        chmod +x /etc/rc.d/rc.local

        touch /var/tmp/.kc1
        echo "Applying break....        DONE"
        echo "Your system has been modified."

}

fix() {
        # checking to make sure we have the .GOOD.crt and .GOOD.key saved from the break
        if [[ -f /var/tmp/.kc1 ]]; then
                mount -o remount,rw /
                 lvextend -L 1G /dev/mapper/application-app

                 mount -a
                 reboot

                reboot
                echo "Applying fix.........     SUCCESS"
        else
                echo "It seems like break was not run successfully.  Run $0 break first."
                exit 7
        fi
}


grade() {

        STATUS="failed"
        echo "Grading.  Please wait."

        if  ! [ -f /var/tmp/.kc1 ]
        then
                echo "It seems like break was not run successfully.  Please relaunch the instance."
                exit 7
        fi



        [ ! -z $(systemctl list-units --type target --state active | grep -o $(systemctl get-default)) ] 2>> /dev/null
        var1=$?


        [[ ! -z $(pvscan 2>&1 | grep -o "PV /dev/vdb    VG application") ]] 2>> /dev/null
        var2=$?



        [ ! -z "$(blkid|grep -o /dev/mapper/application-app)" ] 2>> /dev/null
        var3=$?


        [ ! -z $(lvs 2>> /dev/null | awk '($2=="application"){print $3}' | grep -o a)  ]
        var4=$?


        [ ! -z "$(grep /dev/mapper/application-app /proc/self/mounts|grep -o "/app")" ] &>> /dev/null
        var5=$?



        [ $(awk '/^menuentry.*{/,/^}/' /boot/grub2/grub.cfg | awk -vRS="\n}\n" -vDEFAULT="$((default+1))" 'NR==DEFAULT' | grep -o '/vmlinuz-.*' | awk '{print $1}' | cut -c 10-) == $(uname -a | awk '{print $3}') ] &>> /dev/null
        var6=$?



        [ -z "$(egrep "root|swap|app" /tmp/fstab-capture.out 2>&1|egrep -o "#|No")" ] &>> /dev/null
        var7=$?


        [ ! -z "$(grep /dev/mapper/application-app /tmp/mounts-capture.out 2>/dev/null|grep -o "/app")" ] &>> /dev/null
        var8=$?

        [ -z "$(journalctl -xb | grep -o emergency | sort -u)" ] &>> /dev/null
        var9=$?


        if [[ $var1 == "0" && $var2 == "0" &&  $var3 == "0"  && $var4 == "0"  &&  $var5 == "0" && $var6 == "0"  && $var7 == "0" && $var8 == "0"  &&  $var9 == "0"  ]]
        then
                STATUS="success"
        fi

        # end your grading code here

        if [[ $STATUS == "success" ]]
        then
                echo "Success."
                echo "${bold}COMPLETION CODE: CANTRHEL79BOOTUSER${normal}"
        else
                echo "Sorry.  There still seems to be a problem"
        fi

}

case "$1" in
        break)
                break
                ;;
        grade)
                grade
                ;;
  #     fix)
  #             echo "This will revert the changes made by break."
  #              read -p "Are you sure you want to continue (y/n)? " ANSWER
  #              if [[ "$ANSWER" == "y" ]]; then
  #                      fix
  #              else
  #                      echo "Exiting without making a change."
  #              fi
  #              ;;
        *)
                echo $"Usage: $0 {grade}"
                exit 2
esac


# ending
logger -p local0.notice "Completed $0 with option $@ successfully"
echo "Completed $0 with option $@ successfully"
exit 0
