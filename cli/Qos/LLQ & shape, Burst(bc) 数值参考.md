> *IPP5 burst of CE-1 = <ce1_ipp5_bw> / 1000 x 31.25 in bytes, smallest value is 50000*

**金流量的配置是使用LLQ (具有优先级支持的低延迟排队)**

**priority 后面填kbps, 比如客户订购b/w是1M, 那么参数就是: 1024 or 1000**

```
priority 1024
//
priority 1000
```

**police cir 后面填bps, 比如客户订购b/w是1M, 那么参数就是: 1024000 or 1000000**

bc(burst) *速算: 1024000/1000*31.25=32000, 结果小于5w, 配置就输入50000, 如果大于5w, 配置就输入真实计算出来的结果.* 

```
police cir 1024000 bc 50000
conform-action transmit
exceed-action drop

///
完整的配置: 
!
policy-map child-egress-policy
 class gold-ipp5
  set precedence 5
  priority 1024
  police cir 1024000 bc 50000
   conform-action transmit 
   exceed-action drop
!
```

> IPP3 burst of CE-1 = <ce1_ipp3_bw> / 1000 x 31.25 in bytes, smallest value is 50000

银流量的配置是使用CBWFQ (基于类的加权公平排队)

**bandwidth后面填kbps, 如客户订购带宽是1M, 那么参数就是: 1024 or 1000**

*bc(burst), CBWFQ这类的bc一般不用配置. 保持思科自动的默认参数即可*

```
class silver-ipp3
   set precedence 3
   bandwidth 1000
   random-detect
```

> IPP1 burst of CE-1 = <ce1_ipp1_bw> / 1000 x 31.25 in bytes, smallest value is 50000

铜流量的配置是使用CBWFQ (基于类的加权公平排队)

**bandwidth后面填kbps, 如客户订购带宽是1M, 那么参数就是: 1024 or 1000**

*bc(burst), CBWFQ这类的bc一般不用配置. 保持思科自动的默认参数即可*

```
class class-default
  set precedence 1
  bandwidth 1024
  random-detect
```

> class-default shape & burst = <ce1_total_bw> X 0.004

最后以上金银铜三组class中合计的带宽是4M, 调用到默认的出接口police-map中, shape的参数也是4M.  

- shape average后面填bps, 4M带宽填: 4096000, or 4000000, shape average *bc(burst), = 4000000*0.004*

```
policy-map egress-policy
 class class-default
  shape average 4096000 160000
   service-policy child-egress-policy
```

- 大多数情况下, shape的bc参数都可以不用填写, 保持思科自动的默认参数即可

```
policy-map egress-policy
 class class-default
  shape average 4096000
   service-policy child-egress-policy
```
