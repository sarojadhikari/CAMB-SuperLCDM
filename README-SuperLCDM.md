## Cobaya + New CAMB SuperLCDM with (A0 + eps)

This document describes the changes made to `camb` and `cobaya` to run with `A0` and `eps` SuperLCDM parameters.

1. First, add parameters `A0` and `eps` parameters to the `fortran` code:
    * In `model.f90`: 
    ```fortran
      read(dl) :: A0 = 0._dl
      read(dl) :: eps = 0._dl
    ```
    * In `camb.f90`:
    ```python
      call Ini%Read('A0', P%A0)
      call Ini%Read('eps', P%eps)
    ```
2. Second, add the parameters in the `python` code:
    * In the `CAMBparams` class in `model.py`, add the relevant parameters when others are initialized:
    ```python
      def set_cosmology(self, .... , A0=0.0, eps=0.0):
        ...
        ...
        self.A0 = A0
        self.eps = eps
     ```
     * In `model.py`, add `A0` and `eps` to `_fields_` list in class `CAMBparams`:  
   ```python
      _fields_ = [...,("A0", c_double, "super-lcdm A0 scaling")
                     ,("eps", c_double, "super-lcdm eps factor"),...]
   ```
     * Optionally, if `camb` is used with `cosmomc`, add the aparameters to `set_params_cosmomc` in `camb.py`.
 3. Third, make changes to `get_cmb_power_spectra` in `CAMBdata` class of `results.py`:  
     * To make sure that `Cl(ns)` is called later by `camb`: first in `cobaya` make sure that the 
       first call to `camb` is made using `Cl(ns+eps)`, then in `get_cmb_power_spectra`, the 
       second call can be made using `Cl(ns)`. For `eps=A0=0`, we get back the original; this can 
       be made sure by setting `eps=0` whenever `A0=0`. Implementation can be seen in the following two points:
     * In `cobaya.theoreis.camb.camb.calculate`, set:
          ```python
          if (params.A0 != 0):
            args["ns"]=args["ns"]+params.eps
          ```
     * In `camb.results.get_cmb_power_spectra`, add:
       ```python
          Ptotal = P
          if (params is None) and (self.Params.A0 !=0.0):
            nsp = self.Params.InitPower.ns - self.Params.eps
            As = self.Params.InitPower.As
            self.Params.InitPower.set_params(As=As, ns=nsp)
        
            self.power_spectra_from_transfer()
            
            P2 = {}
            
            for spectrum in spectra:
              P2[spectrum] = getattr(self, 'get_' + spectrum + '_cls')(lmax, 
                                                                      CMB_unit=CMB_unit,
                                                                      raw_cl=raw_cl)
            Ptotal["total"] = P2["total"] + self.Params.A0 * P["total"]
        
          return Ptotal
       ```
       
