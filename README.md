# graylog mongodb fluent elasticsearch deploy on kubernetes

部署方法:

1.提前下载好image镜像和yaml文件放到kubernetes-graylog/文件下
2.上传所有文件到/root目录下

3.#在master节点执行
cat > /etc/sysctl.d/k8s.conf <<EOF
vm.max_map_count = 655360
EOF

sysctl -p /etc/sysctl.d/k8s.conf

for m in `ls /root/kubernetes-graylog|grep tar`;do docker load -i /root/kubernetes-graylog/$m;done

#分发镜像到各个K8S NODE上
for i in `seq 3 10`;do 
    IP=xxx.xxx.xxx.$i
    scp -r /root/kubernetes-graylog $IP:/root/
    scp /etc/sysctl.d/k8s.conf $IP:/etc/sysctl.d/
    ssh $IP sysctl -p /etc/sysctl.d/k8s.conf
    for m in `ls /root/kubernetes-graylog|grep tar`;do ssh $IP docker load -i /root/kubernetes-graylog/$m;done
    echo $IP' All Setup Done!>-------------------------------------------'
done

cd ~/kubernetes-graylog

kubectl create -f .

#等待启动成功后查看日志

kubectl logs -f -n kube-system graylog-master-0
kubectl logs -f -n kube-system elasticsearch-0

#启动成功后访问
http://xxx.xxx.xxx.xxx:31300/

#删除
cd ~/kubernetes-graylog

kubectl delete -f .
