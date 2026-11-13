# Anti-Corruption Layer

An Anti-Corruption Layer (ACL) translates between an external or legacy model and the model used inside a Bounded Context. It protects the local model from adopting foreign terminology, data structures, and assumptions.

An ACL may translate requests, responses, messages, and errors. It should express the local model rather than merely pass data through. Keep the translation boundary explicit and test it against representative legacy behavior.
