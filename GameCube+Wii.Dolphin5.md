# Dolphin 5 for GameCube/Wii

## Pattern
* In `state`, attach to Dolphin
* In `update`, validate any existing found memory. If not found:
  - Iterate through Dolphin's mapped regions of memory:
    + for a region of size 32MB
    + beginning with the desired game ID
  - Set up watchers when memory is found

## Example
```c#
// Attach to Dolphin
state("Dolphin") {
}

// Set up default values
startup {
  vars.BaseAddress = IntPtr.Zero;
  vars.Watchers = new MemoryWatcherList();
}


update {
  
  // If no existing found memory, attempt to find memory
  if (vars.BaseAddress == IntPtr.Zero) {
    // What game ID are we looking for (this is Metal Gear Solid The Twin Snakes)
    string targetGameId = "GGSEA4";
  
    // Iterate through all memory regions
    foreach (var page in game.MemoryPages(true))
    {
      // Check for mapped region size 32MB
      if ((page.RegionSize != (UIntPtr)0x2000000) || (page.Type != MemPageType.MEM_MAPPED)) {
        continue;
      }

      // Check game ID matches
      string foundGameId = memory.ReadString((IntPtr)page.BaseAddress, 6);
      if (!foundGameId.Equals(targetGameId)) {
        continue;
      }
      
      // Memory has been found
      vars.BaseAddress = page.BaseAddress;
      print("Game memory found at address " + vars.BaseAddress.ToString("X"));
      
      // Set up watchers
      vars.Watchers = new MemoryWatcherList() {
        // The game ID is used later to check the base address is still valid
        new StringWatcher(vars.BaseAddress, 6) { Name = "GameId", FailAction = MemoryWatcher.ReadFailAction.SetZeroOrNull },
        
        new MemoryWatcher<byte>(IntPtr.Add(vars.BaseAddress, 0x123456)) { Name = "ByteValue" },
        new StringWatcher(IntPtr.Add(vars.BaseAddress, 0x5a348c), 7) { Name = "StringValue" }
      };
      
      break;
    }
    
    return false;
  }
  
  // Update all values
  vars.Watchers.UpdateAll(game);
  
  // Check that the base address is still valid and pointing at the same game
  if (vars.Watchers["GameId"].Changed) {
    vars.BaseAddress = IntPtr.Zero;
    vars.Watchers = new MemoryWatcherList();
    print("Game memory location lost");
    return false;
  }
  
  // Add game-specific autosplitter logic here, e.g.
  if (vars.Watchers["ByteValue"].Current == 1) {
  }
  
  return true;
}


// Example action blocks...
start {
  return ((vars.Watchers["ByteValue"].Changed) && (vars.Watchers["ByteValue"].Old == 0));
}

split {
  return (vars.Watchers["StringValue"].Changed);
}

reset {
  return ((vars.Watchers["ByteValue"].Changed) && (vars.Watchers["ByteValue"].Current == 0));
}
