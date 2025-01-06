```c++
      section("verify fri layers", {
        store("tmp1_proof_ptr", expr("proof_ptr")),
        store("decommitment_it", expr("tmp1_proof_ptr->fri_queries.decommitments.begin")),
        store("fri_layer_it", expr("fri_layers.begin")),
        muladd(var("fri_layer_count") = x, def_const(fri_layer_info_t.size()), expr("fri_layers.begin"), expr("fri_layers.end")) << locals(1),
        store("query_index_mask_base", expr("eval_domain_size")),
        do_while(expr("fri_layer_count"), {
          print_time("start checking fri layer = "),
          fixed_select(expr("fri_layer_it->merkle_hash.count"), {
            section("fri layer is committed", {
              assert_eq(expr("values_size"), $1),
              let("next_hash", hash_ptr),
              push,
              merkle_path::verify_vx_all({expr("vx"), expr("hash"), *expr("decommitment_it"), expr("fri_query_count"), expr("fri_layer_it->merkle_hash")}) = {"hash", "vx"},
              mov(expr("hash"), expr("next_hash")),
              load_frame(1),
              vx::div_all({expr("vx"), expr("bitreverse_index"), $16, expr("fri_query_block_count")}) = {"_", "path_value_index", "vx"},
              pop,
              store3({"hash", "decommitment_it", "values_begin"}, {expr("next_hash"), expr("decommitment_it"), expr("values_begin")}),
              store3({"values_size", "fri_layer_it", "query_index_mask_base"}, {expr("values_size"), expr("fri_layer_it"), expr("query_index_mask_base")}),
              store3({"fri_offset", "fri_generator", "fri_shift"}, {expr("fri_offset"), expr("fri_generator"), expr("fri_shift")}),

              // we need to adjust the offset to skip fri_query_count's for each offset
              vx::ALLOC_SIZED_PTR(),
              vadd3({expr("path_value_index"), &expr("fri_query_count"), var("path_value_index") = cast(x, sized_strided_ptr_t)}, {vx::MUL_ANY_OP2, vx::STRIDE_0, $0}, {*(expr("vx") + 0), *(expr("vx") + 1), *(expr("vx") + 2)}) << locals(1),
              // add(&$32, vx::MUL_STRIDE_0, *(expr("vx") + 0)),
              // mov(expr("path_value_index"), *(expr("vx") + 1)),
              // NATIVE([] (auto ctx) {ctx.memory->set(ctx.cnt + Felt{x.val_}, ctx.memory->alloc() + vx::STRIDE_1_v);}),
              // mov(var("path_value_index") = cast(x, strided_vec32_ptr_t), *(expr("vx") + 2)) << locals(1),
              store("vx", expr("vx") + 3),

              vx::ALLOC_SIZED_PTR(),
              vadd3({expr("path_value_index"), &expr("decommitment_it->values.begin"), var("tmp") = cast(x, sized_strided_ptr_t)}, {vx::ANY_OP2, vx::STRIDE_0, $0}, {*(expr("vx") + 0), *(expr("vx") + 1), *(expr("vx") + 2)}) << locals(1),
              // mov(expr("path_value_index"), *(expr("vx") + 0)),
              // add(&expr("decommitment_it->values.begin"), vx::STRIDE_0, *(expr("vx") + 1)),
              // NATIVE([] (auto ctx) {ctx.memory->set(ctx.cnt + Felt{x.val_}, ctx.memory->alloc() + vx::STRIDE_1_v);}),
              // mov(var("tmp") = cast(x, strided_vec32_ptr_t), *(expr("vx") + 2)) << locals(1),
              store("vx", expr("vx") + 3),

              NATIVE([] (auto& ctx) {ctx.memory->set(ctx.cnt + Felt{x.val_}, ctx.memory->alloc());}),
              vadd3({expr("tmp"), &expr("0_to_31"), var("path_value_it") = cast(x, ptr_of(felt_t))}, {vx::ANY_FLAG, vx::STRIDE_1, vx::STRIDE_1}, {*(expr("vx") + 0), *(expr("vx") + 1), *(expr("vx") + 2)}) << locals(1),
              assert_address(expr("path_value_it")),
              // mov(expr("tmp"), *(expr("vx") + 0)),
              // add(&expr("0_to_31"), vx::STRIDE_1, *(expr("vx") + 1)),
              // NATIVE([] (auto ctx) {ctx.memory->set(ctx.cnt + Felt{x.val_}, ctx.memory->alloc() + vx::STRIDE_1_v);}),
              // mov(var("path_value_it") = cast(x, strided_vec32_ptr_t), *(expr("vx") + 2)) << locals(1),
              store("vx", expr("vx") + 3),

              push,
              load_frame(1),
              vx::load_all({expr("vx"), expr("path_value_it"), $1, expr("fri_query_block_count")}) = {"expected_value", "vx"},
              pop,
```
