# 06. Industrial IoT: Learning Topology

> "Industrial control systems where the topology learns anomaly patterns across millions of sensors in real time."

In a massive factory or a smart city, the relationships between sensors aren't static. They change. A vibration in Pump A might start correlating with a temperature spike in Valve B only when the external temperature is high.

## The Scenario
We are monitoring a nuclear power plant.
We have 1 million sensors.
We need to detect anomalies not just in *values*, but in *relationships*.

## The Implementation

### 1. The Dynamic Graph
Each sensor is a node. Edges represent correlations.

```rust
use ruvector_graph::{GraphDB, NodeBuilder};

fn update_topology(graph: &mut GraphDB, sensor_data: Stream<SensorReading>) {
    // Real-time stream processing
    for reading in sensor_data {
        // 1. Update the node's current state vector
        let node = graph.get_mut_node(&reading.sensor_id).unwrap();
        node.update_vector(reading.feature_vector);
        
        // 2. Dynamic Edge Creation (Self-Learning Topology)
        // If this sensor's vector aligns with another sensor's vector, strengthen the link.
        let correlated_sensors = graph.vector_search(&reading.feature_vector, 5).unwrap();
        
        for neighbor in correlated_sensors {
            if neighbor.score > 0.9 {
                // Strong correlation: Create or strengthen edge
                graph.upsert_edge(
                    &reading.sensor_id, 
                    &neighbor.id, 
                    neighbor.score // Weight = Correlation
                ).unwrap();
            } else {
                // Weak correlation: Decay edge
                graph.decay_edge(&reading.sensor_id, &neighbor.id, 0.1).unwrap();
            }
        }
    }
}
```

### 2. Topological Anomaly Detection
An anomaly isn't just a high temperature. It's when the *structure* of the system breaks.
e.g., "Pump A usually correlates with Valve B. Today, it correlates with Turbine C. **ALARM.**"

```rust
fn detect_structural_anomaly(graph: &GraphDB) {
    // Use Graph Neural Network (GNN) to embed the topology itself
    let topology_embedding = graph.compute_graph_embedding();
    
    // Compare with "normal" state
    let distance = distance(topology_embedding, NORMAL_STATE_VECTOR);
    
    if distance > THRESHOLD {
        trigger_alarm("System topology has shifted significantly!");
    }
}
```

## The Vision
The system builds its own map of reality. You don't tell it how the factory works; it watches, learns the correlations, and alerts you when the "shape" of the operation changes.

## Next
From physical flows to money flows. Go to **[07_financial_market.md](./07_financial_market.md)**.
