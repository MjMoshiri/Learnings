# Blockchain (Java)

Hand-rolled blockchain in Java to apply what I learned from the HyperSkill Java track. SHA-256 hashes for blocks, serialization for save/load, adjustable proof-of-work difficulty, singleton chain with synchronized validation. Eight miner threads racing to find blocks.

Point was concurrency and serialization, not crypto. Threading the miners forced me to think about shared state, race conditions, and the Singleton pattern in a setting where it actually made sense.
