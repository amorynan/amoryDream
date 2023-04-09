![img.png](img.png)

CK 关于序列化的调用栈分析
写入数据：
```cgo
// step1. get storage engine (MergeTreeBlockOutputStream)
/** Writes data to the specified table and to all dependent materialized views.
  */
PushingToViewsBlockOutputStream(const StoragePtr & storage_,
    const StorageMetadataPtr & metadata_snapshot_,
    ContextPtr context_,
    const ASTPtr & query_ptr_,
    bool no_destination) {
    
    // from table_id 
     Dependencies dependencies = DatabaseCatalog::instance().getDependencies(table_id);
     if (!dependencies.empty()) {
        insert_context = Context::createCopy(context);
    }
    for (const auto & database_table : dependencies) {
        auto dependent_table = DatabaseCatalog::instance().getTable(database_table, getContext());
        auto dependent_metadata_snapshot = dependent_table->getInMemoryMetadataPtr();

        ASTPtr query;
        BlockOutputStreamPtr out;
        
        out = std::make_shared<PushingToViewsBlockOutputStream>(
                dependent_table, dependent_metadata_snapshot, insert_context, ASTPtr());
        views.emplace_back(ViewInfo{std::move(query), database_table, std::move(out), nullptr, 0 /* elapsed_ms */}); 
    }
    
       /// Do not push to destination table if the flag is set
    if (!no_destination) {
        BlockOutputStreamPtr output = storage->write(query_ptr, storage->getInMemoryMetadataPtr(), getContext());
        ReplicatedMergeTreeBlockOutputStream* replicated_output = dynamic_cast<ReplicatedMergeTreeBlockOutputStream *>(output.get());
    }
    }


```