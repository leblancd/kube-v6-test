# kube-v6-test
This page describes a suite of Kubernetes e2e tests that can be used for Kubernetes IPv6 Continuous Integration (CI) testing, or for testing IPv6 networking conformance on an IPv6-only, multi-node Kubernetes cluster.

The test cases that are included in this suite are all "out-of-the-box" Kubernetes e2e test cases, that is, they are available upstream (although some require fixes from pull requests that have not yet been merged, as described below). Running this test suite is therefore a matter of providing the right test case filtering through the use of "--ginkgo.focus" and "--ginkgo.skip" regular expressions on the command line, as described in the [Kubernetes e2e test documentation](https://github.com/kubernetes/community/blob/master/contributors/devel/e2e-tests.md#end-to-end-testing-in-kubernetes), as well as a "--num-nodes=2" flag to run the Kubernetes services e2e tests that require 2 or more worker nodes.

## The Evolving Test Suite
It is expected that the list of test cases that are included in this suite will grow over time to improve test coverage and make the testing more comprehensive. Some things to consider before adding a test case:
- Does the test work in an IPv6 cluster (is it debugged)?
- Is it a meaningful test of IPv6 functionality?
- Is it fairly quick to run? (important for keeping CI test queues reasonable)

## Disabling IPv4-Specific Test Cases for IPv6 Testing
If there is an IPv4-specific Kubernetes e2e networking test case that should be excluded from testing on an IPv6-only cluster, then the test should be marked as being an IPv4 test case by adding the following tag in the test description:
```
[Feature:Networking-IPv4]
```
For example, there is a 'ping 8.8.8.8' test that has been disabled for IPv6 testing as follows:
```
It("should provide Internet connection for containers [Feature:Networking-IPv4]", func() {
```
Any tests with this tag can be excluded from e2e testing by including "IPv4" as part of the --ginkgo.skip regular expression on the e2e test command line (see "e2e Test Command Line" below).


## Disabling IPv6-Specific Test Cases for IPv4-only Testing
Conversely, if there is an IPv6-specific Kubernetes e2e networking test case that should be excluded from testing on an IPv4-only cluster, then the test case should be marked with the following tag in the test description:
```
[Feature:Networking-IPv6][Experimental]
```
For example:
```
It("should provide Internet connection for containers [Feature:Networking-IPv6][Experimental]", func() {
```
Any test with this tag can be excluded from e2e testing by including "IPv6" as part of the --ginkgo.skip regular expression on the e2e test command line (see "e2e Test Command Line" below).

## Required PRs
There are still some outstanding, not-yet-merged Kubernetes pull requests that are required for running the test suite:

Kubernetes operational code:
- [PR #47621](https://github.com/kubernetes/kubernetes/pull/47621)
- [PR #50929](https://github.com/kubernetes/kubernetes/pull/50929)
- [PR #50478](https://github.com/kubernetes/kubernetes/pull/50478)
- [PR #52033](https://github.com/kubernetes/kubernetes/pull/52033)
- [PR #53555](https://github.com/kubernetes/kubernetes/pull/53555)

Kubernetes e2e test code:
- [PR #52748](https://github.com/kubernetes/kubernetes/pull/52748)
- [PR #53384](https://github.com/kubernetes/kubernetes/pull/53384)
- [PR #53389](https://github.com/kubernetes/kubernetes/pull/53389)
- [PR #53531](https://github.com/kubernetes/kubernetes/pull/53531)
- [PR #53569](https://github.com/kubernetes/kubernetes/pull/53569)

## Running the IPv6 Multi-node e2e Test Suite

#### If you haven't already done so, copy the kubernetes config file and the kubectl binary from your kube-master to your build node

The following assumes that you have password-less access to kube-master for user "kube" (but no scp access for root):
```
#!/bin/bash

KUBE_USER=kube
KUBE_MASTER=kube-master
echo ssh into $KUBE_MASTER and copy Kubernetes config and kubectl to $KUBE_USER home directory
ssh $KUBE_USER@$KUBE_MASTER << EOT
mkdir -p /home/kube/.kube
sudo yes | sudo cp -f /etc/kubernetes/admin.conf /home/$KUBE_USER/.kube/config
sudo chown $(id -u):$(id -g) /home/$KUBE_USER/.kube/config
sudo yes | sudo cp -f /bin/kubectl /home/$KUBE_USER/.kube
EOT

echo scp kubernetes config from $KUBE_MASTER to /home/$KUBE_USER/.kube/config
mkdir -p $HOME/.kube
scp $KUBE_USER@$KUBE_MASTER:/home/$KUBE_USER/.kube/config $HOME/.kube
chown $(id -u):$(id -g) $HOME/.kube/config
scp $KUBE_USER@$KUBE_MASTER:/home/$KUBE_USER/.kube/kubectl $HOME/.kube
sudo cp $HOME/.kube/kubectl /bin/kubectl
```
#### Confirm that you can connect from your build server to the Kubernetes API serve using kubectl:
```
some-user@build-server:~$ kubectl get nodes
NAME            STATUS    ROLES     AGE       VERSION
kube-master     Ready     master    6d        v1.9.0-alpha.0.ipv6.0.1+23df37a5a1b7d7-dirty
kube-minion-1   Ready     <none>    6d        v1.9.0-alpha.0.ipv6.0.1+23df37a5a1b7d7-dirty
kube-minion-2   Ready     <none>    6d        v1.9.0-alpha.0.ipv6.0.1+23df37a5a1b7d7-dirty
some-user@build-server:~$ 
```
If you don't get a response, check that you've copied the Kubernetes config file correctly from the kube-master  to $HOME/.kube/config (previous step), and check that you have the required routes from your build node to the Kubernetes API service IP.

#### You should also be able to curl the Kubernetes API server:
```
some-user@build-server:~$ curl -g [fd00:1234::1]:443 | od -c -a
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100    14    0    14    0     0    265      0 --:--:-- --:--:-- --:--:--   269
0000000 025 003 001  \0 002 002  \n
        nak etx soh nul stx stx  nl
0000007
some-user@build-server:~$
```

#### Running the tests
The e2e tests can be run as follows:
```
export KUBECONFIG=/home/openstack/.kube/config
export KUBE_MASTER=local
export KUBE_MASTER_IP="[fd00:1234::1]:443"
export KUBERNETES_CONFORMANCE_TEST=n
cd $GOPATH/src/k8s.io/kubernetes
go run hack/e2e.go -- --provider=local -v --test --test_args="--host=https://[fd00:1234::1]:443 --ginkgo.focus=Networking|Services --ginkgo.skip=IPv4|Networking-Performance|Federation|preserve\ssource\spod --num-nodes=2"
```

An explanation of some of the fields used in this command set:
```
- Kubernetes API Service IP:  fd00:1234::1
- INCLUDE test cases with the following phrases/words in their descriptions:
  - "Networking"
  - "Services"
- But EXCLUDE test cases with the following phrases/words in their descriptions:
  - "IPv4"
  - "Networking-Performance"
  - "Federation"
  - "preserve source pod"
- Number of worker nodes to use for testing: 2 (Min required for some service tests)
```

## Included Test Cases

## NOTE: THE LIST BELOW NEEDS UPDATING: ABOUT 14 PASSING TEST CASES ARE MISSING

#### Network Connectivity Test Cases:

- should function for node-pod communication: udp [Conformance]
- should function for node-pod communication: http [Conformance]
- should function for intra-pod communication: udp [Conformance]
- should function for intra-pod communication: http [Conformance]
- should provide unchanging, static URL paths for kubernetes api services
- should provide Internet connection for containers [Feature:Networking-IPv6][Experimental]

#### Services Test Cases:

- should function for endpoint-Service: http
- should function for node-Service: http
- should function for pod-Service: udp
- should update endpoints: udp
- should function for endpoint-Service: udp
- should function for node-Service: udp
- should check kube-proxy urls
- should update nodePort: http [Slow]
- should function for pod-Service: http
- should update endpoints: http
- should update nodePort: udp [Slow]

## Wish List (Failing Tests That Would be Good to Have Fixed)

- [sig-network] DNS [It] should provide DNS for services [Conformance] 
- [sig-network] Services [It] should preserve source pod IP for traffic thru service cluster IP 
- [sig-network] ServiceLoadBalancer [Feature:ServiceLoadBalancer] 
  should support simple GET on Ingress ips

