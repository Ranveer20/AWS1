#### AWS VM SWAP Space check #####

vmlist=$(aws ec2 describe-instances --query 'Reservations[*].Instances[*].[PublicDnsName]' --output text )
for h in $vmlist; do
echo $h
swap=$(ssh -i "ranveer1vm.pem" ec2-user@$"$h"  "grep SwapTotal /proc/meminfo" | grep -wo "0")
if [ $swap -eq 0 ]; then
echo "No SWAP Memory " $h
echo $h >> no_swap_vm.txt
else
echo "$h is DOWN"
echo $h >> swap_exist_vm.txt
fi
done

#### AWS VM create and delete #####

while read vm
do

aws ec2 describe-instances --filter "Name=dns-name,Values=$vm" --query 'Reservations[].Instances[].[ [Tags[?Key==`Name`].Value][0][0],InstanceId,InstanceType,SubnetId,KeyName,ImageId ]' --output text > awsvm.txt
aws ec2 describe-instances --filter "Name=dns-name,Values=$vm" --query 'Reservations[].Instances[].[ [Tags[?Key==`Name`].Value][0][0],InstanceId,InstanceType,SubnetId,KeyName,ImageId ]' --output text >> awsvmlist.txt

awsTag=$(awk '{print $1}' awsvm.txt)
awsID=$(awk '{print $2}' awsvm.txt)
awsTYPE=$(awk '{print $3}' awsvm.txt)
awsSUB=$(awk '{print $4}' awsvm.txt)
awsKEY=$(awk '{print $5}' awsvm.txt)
awsIMG=$(awk '{print $6}' awsvm.txt)

echo "$awsTag","$awsID","$awsTYPE","$awsKEY"


   echo "AWS Vm $awsTag is going to Delete"

aws ec2 terminate-instances --instance-ids "$awsID"

sleep 5m

aws ec2 run-instances --image-id "$awsIMG" --count 1 --instance-type "$awsTYPE" --key-name "$awsKEY" --tag-specifications 'ResourceType=instance,Tags=[{Key=Name,Value='$awsTag'}]'

done < no_swap_vm.txt
