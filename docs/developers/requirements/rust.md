# Requirements >> Rust ||30

From experience, we know that one of the major blockers while learning holochain is rust. This is one of the reasons that the gym exists, to have nice interactive exercises without needing to do any coding at all.

Learning rust is outside the scope of the gym by itself. However, here you can find a fabulous list of resources to start learning rust specifically targeted at holochain beginners:

- [Learning Rust Resources](https://forum.holochain.org/t/learning-rust/3820)

We also will be providing some small rust tips along the way in some exercises to help you unblock yourself. We hope that you can overcome this obstacle and start having fun with holochain itself! Let us know if there is something that we could add to improve the experience.

## Rust analyzer

```js script
import "@rocket/launch/inline-notification/inline-notification.js";
```

<inline-notification type="tip" title="Recommended">

We highly recommend installing the `rust-analyzer` VSCode extension and opening your projects from a folder which has a `Cargo.toml` in it.  
The experience of writing Rust this way is much more pleasant and allows for a much faster learning curve.

</inline-notification>

### Gotcha's

- For rust-analyzer to work, you need to have rust installed on outside the nix-shell, on your host system. You can follow [these instructions](https://www.rust-lang.org/tools/install).
  - If you are using WSL, you should install rust inside the linux subsystem, and not the Windows host.