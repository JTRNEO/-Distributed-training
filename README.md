# -Distributed-training
Use kubernetes and horovod.it has three object detection models include pytorch-panet,mxnet-sniper and tensorflow2-yolov3.

Now it's easy to use Rancher to set up a kubernetes cluster for yourself!

you can follow xxx to build.(if you only have one PC.you can set a cluster that only have one node to test.)

when you have your own cluster.

install helm.

prepare work are finished until now.

######################for model training.

make your own models docker image use the example Dockerfile.

Note:if your model use the  framework that needs compile manually like sniper.

you can do like this :

make your models docker images that include all requirements except nccl2,openmpi and horovod.

then we run the container, attach the container, compile the package of the framework ,install nccl2,openmpi and horovod 

manually.

commit the container to new image.

warning :do not use ubuntu18.04 as your basic image. ssh while failed if you use ubuntu18.04.

warning :if your model is using tensorflow 2.0 beta.  before you install horovod , add the command  like below:

              ln -s tensorflow_framework.so.2 libtensorflow_framework.so.1
            
create presitentvolume

kubectl apply -f xxx.yaml

create presitentvolumeclaim

kubectl apply -f xxx_claim.yaml

helm fetch --untar stable/horovod

cd /horovod/

modify /horovod/values.yaml like example /horovod-sniper/values.yaml

modify /horovod/templates/job.yaml like example /horovod-sniper/templates/job.yaml

modify /horovod/templates/statefulset.yaml like /horovod-sniper/templates/statefulset.yaml

run the command like below:
             
                     helm install --name xxx  /your horovod path
                     
get into one of the pods.(master pod)

                      kubectl get pod
                      kubectl exec -it PodName /bin/bash
                      
when you get into the pod.

modify your code follow this repo:

https://github.com/horovod/horovod.git
steps:

1.Run hvd.init().
2.Pin a server GPU to be used by this process using config.gpu_options.visible_device_list. With the typical setup of one GPU per process, this can be set to local rank. In that case, the first process on the server will be allocated the first GPU, second process will be allocated the second GPU and so forth.
3.Scale the learning rate by number of workers. Effective batch size in synchronous distributed training is scaled by the number of workers. An increase in learning rate compensates for the increased batch size.
4.Wrap optimizer in hvd.DistributedOptimizer. The distributed optimizer delegates gradient computation to the original optimizer, averages gradients using allreduce or allgather, and then applies those averaged gradients.
5.Add hvd.BroadcastGlobalVariablesHook(0) to broadcast initial variable states from rank 0 to all other processes. This is necessary to ensure consistent initialization of all workers when training is started with random weights or restored from a checkpoint. Alternatively, if you're not using MonitoredTrainingSession, you can simply execute the hvd.broadcast_global_variables op after global variables have been initialized.
5.Modify your code to save checkpoints only on worker 0 to prevent other workers from corrupting them. This can be accomplished by passing checkpoint_dir=None to tf.train.MonitoredTrainingSession if hvd.rank() != 0.

when you finished modified the code

run command like this :

mpirun -np 32 --hostfile /horovod/generated/hostfile --mca orte_keep_fqdn_hostnames t --allow-run-as-root -x NCCL_SOCKET_IFNAME=^lo sh -c 'your training python script'


#######################################

#####################for model inference ###################
TODO:




                       
