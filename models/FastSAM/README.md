The original FastSAM do not support access to its multi-scale features. To obtain them (including the low-level features),
the following changes are required:

### find '/...anaconda_dir.../lib/python3.xx/site-packages/ultralytics/yolo/engine/predictor.py'
     1. add the following after line 244:
```
            ms_feats = preds[-1]
            return ms_feats
```
        
      so that it becomes:
      
```   
            # Inference
            with profilers[1]:
                preds = self.model(im, augment=self.args.augment, visualize=visualize)
            ms_feats = preds[-1]
            return ms_feats
```
            
     2. comment line 252 to line 276 (to disable the default outputs)
     3. comment line 129 (we already did normalization)
     4. comment line 212&213:
     
```
        #if self.args.verbose:
        #    LOGGER.info('')
```
   
### find '/...anaconda_dir.../lib/python3.10/site-packages/ultralytics/nn/modules/head.py'
     1. add after line 91:   
```
      ms_feats = x
      x = x[:3]
```
      
     2. line 99, changes it into:
     
```
      return (torch.cat([x, mc], 1), p) if self.export else (torch.cat([x[0], mc], 1), (x[1], mc, p), ms_feats)
```

### find '/../ultralytics/nn/tasks.py'
     1. from line 76 we change into:

```
        y, dt = [], []  # outputs
        self.save.append(1)
        for m in self.model:
            if isinstance(m, Segment):
                m.f = [15, 18, 21, 1]
            if m.f != -1:  # if not from previous layer
                x = y[m.f] if isinstance(m.f, int) else [x if j == -1 else y[j] for j in m.f]  # from earlier layers
            if profile:
                self._profile_one_layer(m, x, dt)
            x = m(x)  # run
            y.append(x if m.i in self.save else None)  # save output
            if visualize:
                feature_visualization(x, m.type, m.i, save_dir=visualize)
        return x
```
