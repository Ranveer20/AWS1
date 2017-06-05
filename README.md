AWS vm swap Memory check script



swap=$(ssh -i "ranveer1vm.pem" ec2-user@ec2-52-14-71-49.us-east-2.compute.amazonaws.com  " grep SwapTotal /proc/meminfo" | grep -wo "0")

#echo "SWAP is"  $swap

if [ $swap == 0 ]
then
echo "No SWAP Memory " $swap
echo $awsvm >> swap_vm.txt
else
echo "SWAP Memory Exist"
fi
