As a fallback solution, we can use VM-provisioned haproxy to serve as loadbalancer, but this require extra a customer cloud-provider

- watch kubernetes api for any loadbalancer request
- create/configure VM, assign the lb ip address
- configure haproxy based on the kubernetes reqeusts
  - get backend pool for kubernetes reqeust
  - configure a vip for this lb 
- set resonse of the lb address to kubrentes api