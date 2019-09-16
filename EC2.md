# EC2

**T2 (Standard mode) vs. T3 (Unlimited mode) pricing**

> If you’ve used the older T2 family, you’ll know that it operates in Standard Mode. This means that if you exhaust your CPU credits while bursting above baseline, your performance is gradually throttled back to baseline until you can earn more CPU credits.
> T3 is different. By default, it operates in Unlimited mode. In unlimited mode, if an instance runs out of credits, it can continue to burst by accumulating a surplus credit charge. 

Ref: https://blog.cloudability.com/aws-ec2-t3-cost-optimization/
