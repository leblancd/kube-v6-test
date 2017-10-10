# kube-v6-test
This page describes a suite of Kubernetes e2e tests that can be used for Kubernetes IPv6 Continuous Integration (CI) testing, or for testing IPv6 networking conformance on an IPv6-only Kubernetes cluster.

The test cases that are included in this suite are all "out-of-the-box" Kubernetes e2e test cases, that is, they are available upstream (although some require fixes from pull requests that have not yet been merged, as described below). Running this test suite is therefore a matter of providing the right test case filtering through the use of "--ginkgo.focus" and "--ginkgo.skip" regular expressions on the command line, as described in the [Kubernetes e2e test documentation](https://github.com/kubernetes/community/blob/master/contributors/devel/e2e-tests.md#end-to-end-testing-in-kubernetes).

## The Evolving Test Suite
It is expected that the list of test cases that are included in this suite will grow over time to improve test coverage and make the testing more comprehensive. Some things to consider before adding a test case:
- Does the test work in an IPv6 cluster (is it debugged)?
- Is it a meaningful test of IPv6 functionality?
- Is it fairly quick to run? (important for keeping CI test queues reasonable)

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

## e2e Test Command Line

```
go run hack/e2e.go -- --provider=local -v --test --test_args="--host=https://[fd00:1234::1]:443 --ginkgo.focus=Networking --ginkgo.skip=IPv4|Networking-Performance|Internet --num-nodes=2"
```

## Included Test Cases

#### Network Connectivity Test Cases:

- should function for node-pod communication: udp [Conformance]
- should function for node-pod communication: http [Conformance]
- should function for intra-pod communication: udp [Conformance]
- should function for intra-pod communication: http [Conformance]
- should provide unchanging, static URL paths for kubernetes api services

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

