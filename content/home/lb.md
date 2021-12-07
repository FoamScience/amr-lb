+++
weight = 4
+++

{{< slide background-image="images/bg-lb.png" >}}

# Load Balancing

---

### Load-Balance the mesh after topology changes

- Refinement methods generally cause imbalance for static domain decomposition cases.

- The Load-Balancer engine in Foam-Extend can auto-sense processor
  imbalance with respect to cell-count

- It also tries to migrates adjacent cells between processors if possible

---

{{< foamCode "1-3,13|4|5-12" >}}
<span class="hljs-title">dynamicFvMesh</span> loadBalanceDynamicPolyRefinementFvMesh<span class="hljs-punctuation">;</span>
<span class="hljs-type">loadBalanceFvMeshCoeffs</span>
<span class="hljs-punctuation">{</span>
    <span class="hljs-title">imbalanceTrigger</span> <span class="hljs-number">0.2</span><span class="hljs-punctuation">;</span>
    <span class="hljs-title">numberOfSubdomains</span> <span class="hljs-number">4</span><span class="hljs-punctuation">;</span>
    <span class="hljs-title">method</span>          hierarchical<span class="hljs-punctuation">;</span>
    <span class="hljs-type">hierarchicalCoeffs</span>
    <span class="hljs-punctuation">{</span>
        <span class="hljs-title">n</span>       <span class="hljs-punctuation">(</span> <span class="hljs-number">2</span> <span class="hljs-number">2</span> <span class="hljs-number">1</span> <span class="hljs-punctuation">)</span><span class="hljs-punctuation">;</span>
        <span class="hljs-title">delta</span>   <span class="hljs-number">0.001</span><span class="hljs-punctuation">;</span>
        <span class="hljs-title">order</span>   xyz<span class="hljs-punctuation">;</span>
    <span class="hljs-punctuation">}</span>
<span class="hljs-punctuation">}</span>
{{< /foamCode >}}

--- 

{{< slide background-color="white" >}}
### Dynamically load-balance after AMR operations

<video style="display:block; width:85%; margin:0 auto; padding:0;" data-autoplay src="videos/loadbalance.webm"></video>


---
### Load-Balanced Chemistry

What Bulut Tekgul et al. have achieved with **[DLBFoam](https://github.com/Aalto-CFD/DLBFoam)**:

- Load balance the number of **Chemistry ODE problems** across processes ([DOI: 10.1016/j.cpc.2021.108073](https://doi.org/10.1016/j.cpc.2021.108073))
    - Solved independently during each CFD iteration
    - Employ reference solution mapping to (thermo-chemically) similar cells

---

<p style="text-align: center;">An example of DLBFoam run with the Hex-based adaptive refinement engine (courtesy of Bulut, Argonne National Labs)</p>
<video style="scale:0.85; display:block; margin:0 auto; padding:0;" data-autoplay src="videos/dlbfoam.webm"></video>

---

### Load-Balancing for particle simulations

A simplistic approach was adopted in [mdFoam+](https://www.sciencedirect.com/science/article/pii/S0010465517303363)
by S.M. Longshaw et al. to balance particles count

```cpp{7-8|10,13,15|21-22}
if ( Pstream::parRun() )
{
    const scalar& allowableImbalance = maxImbalance_;
        
    //First determine current level of imbalance - do this for all
    // parallel runs, even if balancing is disabled
    scalar nGlobalParticles = cloud_.size();
    Foam::reduce(nGlobalParticles, sumOp<scalar>());
    
    scalar idealNParticles = scalar(nGlobalParticles)/scalar(Pstream::nProcs());
    
    scalar nParticles = cloud_.size();
    scalar localImbalance = mag(nParticles - idealNParticles);
    Foam::reduce(localImbalance, maxOp<scalar>());
    scalar maxImbalance = localImbalance/idealNParticles;
    
    Info << "    Maximum imbalance = " << 100*maxImbalance << "%" << endl;
    
    if( maxImbalance > allowableImbalance && enableBalancing_)
    {   
        // Call reconstructPar and decomposeDSMCLoadBalancePar 
        // on master
    }
}
```
