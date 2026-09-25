# Notes for `desilike` LPNG inference

Current implementation in `desilike` is based on the following formula:

$$
P^s_{XY}(k,\mu) =
{\rm AP} \times {\rm FoG}
\times [b_X(k)+f\mu^2] [b_Y(k)+f\mu^2]
P_m(k)
+ P_{\rm shot}.
$$

## 1. Base Effects

### 1.1 AP effect

AP Jacobian 已经被吸收到 pk_dd 里

```python
jac, kap, muap = self.template.ap_k_mu(k, mu)
pk_dd = jac * _interp_loglog(kap, self.template.k, self.template.pk_dd)
```

In old implement (main branch):

```python
pkmu = jac * fog * (bX + f * muap**2) * (bY + f * muap**2) \
            * interp1d(jnp.log10(kap), np.log10(kin), pk_dd) \
            + sn0 / self.nd
```

### 1.2 FoG 

FoG, or small-scale damping factor, is given by
$$
{\rm FoG}(k,\mu) = \frac{1}{(1 + \sigma_s^2 k^2 \mu^2 / 2)^2},
$$
where $\sigma_s$ is a free parameter and is **depended on the tracers**.

```python            
fog = 1. / (1. + self.sigmas**2 * kap**2 * muap**2 / 2.)**2
pkmu = fog * (b_eff + f * muap**2)**2 * pk_dd
```

### 1.3 Kaiser Effect

Model RSD effect with Kaiser term $b(k)+f\mu^2$ in the formula.

### 1.4 Shot-noise

$$
P_{\rm shot} = \frac{s_{n,0}}{\bar{n}} \delta_{XY}^D = \frac{s_{n,0}}{\bar{n}} \delta_{\ell 0}^D
$$

```python
sn = jnp.array([(ell == 0) for ell in self.ells], dtype='f8')[:, None] * self.sn0 / self._nbar
self.poles = self._to_poles(pkmu) + sn
```

## 2. Scale-dependent bias

For each tracer, the effective bias is given by

$$
b_{\rm eff}(k)=b_1+b_{\phi} f_{\rm NL} \alpha(k),
$$
```python
b_eff = self.b1 + bfnl_loc * alpha
```

### 2.1 $b_{\phi} f_{\rm NL}$ sampling

$$
b_\phi=2 \delta_{\rm cr}\left(b_1-p\right). 
$$

3 methods:
- `b-p`: sampling $p$ and $f_{\rm NL}$, default $\delta_{\rm cr}=1.686$
- `bphi`: sampling $b_{\phi}$ and $f_{\rm NL}$
- `bfnl`: sampling $b_{\phi} f_{\rm NL}$ as a whole


### 2.2 Primordial Potential to Matter 

Define $\alpha(k,z)=\mathcal{M}^{-1}(k,z)$. 

Implemented in `desilike/theories/galaxy_clustering/png.py` function `_alpha_png(...)`.

It is calculated in 2 methods. 

The default one is called `prim`, based on 

$$
\alpha(k,z) = \frac{1}{\mathcal{M}(k,z)} = \sqrt{\frac{P_{\phi}(k)}{P_{\delta \delta}(k,z)}} = \sqrt{\frac{9}{25}\frac{2\pi^2}{k^3}\frac{P_{\rm prim}(k)}{h^3} \frac{1}{P_{\delta\delta}(k,z)}}.
$$

```python
pphi_prim = 9. / 25. * 2. * jnp.pi**2 / k**3 * pk_prim / h**3
return 1. / jnp.sqrt(pk_dd / pphi_prim)
```

Remember the gauge transformation formula: $P_{\phi}(k)=\frac{9}{25} \frac{2 \pi^2}{k^3} \mathcal{P}_{\mathcal{R}}(k)$.

The alternative one is called `transfer`, based on 

$$
\alpha(k, z) \propto \frac{\Omega_m H_0^2}{k^2 T(k) D(z)}.
$$

## Implementation Details

（以下信息来自 ChatGPT 2026-09-25 ，未人工核实。）

desilike 目前（2026年9月）在进行重构，最新分支 refactor-jax 在活跃开发中， main branch 停留在 2026-07-29.

在 refactor-jax 分支中， PNG 相关代码也进行了重构，从 
`desilike/theories/galaxy_clustering/primordial_non_gaussianity.py` 
变成 
`desilike/theories/galaxy_clustering/png.py` 

API 也有变化，如从 `PNGTracerPowerSpectrumMultipoles` 变成 `PNGTracerSpectrum2Poles`

**新版支持对 cosmological parameters 的自动微分，为 PNG + standard cosmology inference 奠定了基础。**

*以上笔记主要基于 refactor-jax 分支的代码，经过人工核实，main 分支的少量提及未经过人工核实。*