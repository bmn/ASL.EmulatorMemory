# ePSXe for PlayStation

## Pattern
* ePSXe 2.0.5 game memory starts at `ePSXe.exe+A82020`
* If targeting ePSXe specifically, you can a state descriptor for simplicity
* If targeting multiple emulators, use this as the base for your watcher definitions

## Example (state descriptor)
```c#
state("ePSXe") {
  byte   ByteValue:   0xA82020, 0x123456;
  string StringValue: 0xA82020, 0x789ABC;
}

// Example split block
split {
  return (current.StringValue != old.StringValue);
}
```

## Example (watchers)
```c#
state("ePSXe") {
}

startup {
  vars.BaseAddress = IntPtr.Zero;
}

init {
  vars.BaseAddress = IntPtr.Zero;
  
  if (game.ProcessName.Equals("ePSXe")) {
    vars.BaseAddress = IntPtr.Add(modules.First().BaseAddress, 0xA82020);
  }
  
  if (vars.BaseAddress != IntPtr.Zero) {
    
    // Set up watchers
    vars.Watchers = new MemoryWatcherList() {
      new MemoryWatcher<byte>(IntPtr.Add(vars.BaseAddress, 0x123456)) { Name = "ByteValue" },
      new StringWatcher(IntPtr.Add(vars.BaseAddress, 0x789ABC), 123) { Name = "StringValue" }
    };
  
  }
}

// Example split block
split {
  return (vars.Watchers["StringValue"].Changed);
}
```
