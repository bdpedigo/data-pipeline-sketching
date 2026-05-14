# data-pipeline-sketching

```mermaid
graph TD
    subgraph Segmented Volume
        imagery[Imagery]
        segmentation[Segmentation]
        meshes[Meshes]
        synapse_detection[Synapse Detection]
        nucleus_detection[Nucleus Detection]
    end

    subgraph Perisoma Pipeline
        fix_meshes[Fix Meshes]
        feature_extraction[Perisoma Feature Extraction]
        fix_meshes --> feature_extraction
    end

    subgraph Outputs
        fixed_perisoma_meshes[Fixed Perisoma Meshes]
        data_table[Perisoma/Nuc Features]
        skeletons_out[Rich Skeletons]
        hks_out[HKS Supermoxel Graph + Features]
        synapse_mesh_map_out[Synapse : Mesh Map]
        perisoma_classifiers[Perisoma Cell Type Classification]
        embedding[Perisoma Embedding]
        synapse_hks_features[Synapse : HKS Features]
        synapse_target_classifications[Synapse Target Classifications]
        dendrite_features[Dendrite Features]
        multifeatures[Multifeatures]
        multifeature_cell_type[Multifeature Cell Type Classification]
        multifeature_embedding[Multifeature Embedding]
    end

    subgraph Dendrite Pipeline
        dc_skeletons[Rich Skeletons]
        dc_dendrite_classification[Dendrite Classification]
        dc_feature_extraction[Dendrite Feature Extraction]
        dc_skeletons -->|dendrite classifier| dc_dendrite_classification
        dc_dendrite_classification --> dc_feature_extraction
    end

    subgraph Derived Features
        level2_features[Level2 Features]
        level2_graph[Level2 Graph]
    end

    subgraph HKS Pipeline
        hks_feature_extraction[HKS Feature Extraction]
        synapse_mesh_mapping[Synapse : Mesh Mapping]
    end

    segmentation --> fix_meshes
    synapse_detection --> feature_extraction
    nucleus_detection --> feature_extraction

    fix_meshes --> fixed_perisoma_meshes
    feature_extraction --> data_table
    data_table -->|perisoma classifier| perisoma_classifiers
    data_table --> embedding

    perisoma_classifiers --> dc_feature_extraction
    dc_feature_extraction --> dendrite_features
    synapse_target_classifications --> dc_feature_extraction
    dendrite_features --> multifeatures
    data_table --> multifeatures
    multifeatures -->|multifeature cell type classifier| multifeature_cell_type
    multifeatures --> multifeature_embedding
    synapse_detection --> dc_skeletons
    dc_skeletons --> skeletons_out

    segmentation --> level2_features
    segmentation --> level2_graph
    level2_features --> dc_skeletons
    level2_graph --> dc_skeletons

    meshes --> hks_feature_extraction
    hks_feature_extraction --> hks_out
    meshes --> synapse_mesh_mapping
    synapse_detection --> synapse_mesh_mapping
    synapse_mesh_mapping --> synapse_mesh_map_out
    hks_out --> synapse_hks_features
    synapse_mesh_map_out --> synapse_hks_features
    synapse_hks_features -->|postsynaptic shape classifier| synapse_target_classifications
```

## Proposal for standardization, portability, reproducibility
- Every node on this chart that takes in data -> outputs data should
    - Be a Python function in a library on PyPI
    - Have a well specified environment (implied by the above)
    - Take in those data objects + config/parameters. Parms can be specified in a file (yaml/toml)?
    - If the function needs something else, update the diagram
- Every classifier on this chart should
    - Be stored on the Huggingface hub with a model card
    - The model card should specify what code generated the features (see above)
- Every data output node on this chart should
    - Be in a standard format per type (TBD what these will be, e.g. parquet/delta for tables, .ply or something for mesh, etc.)
    - Be indexed in the catalog service  
