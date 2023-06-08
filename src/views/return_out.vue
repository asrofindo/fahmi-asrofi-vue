<template>
  <div class="app-container">
    <div class="filter-container">
      <el-input
        v-model="search"
        placeholder="Cari"
        style="width: 200px; margin-right: 10px"
        class="filter-item"
      />

      <el-button
        class="filter-item"
        type="primary"
        icon="el-icon-edit"
        @click="handleCreate"
      >
        Tambah
      </el-button>
      <el-button
        v-waves
        style="margin-right: 20px"
        :loading="downloadLoading"
        class="filter-item"
        type="primary"
        icon="el-icon-download"
        @click="handleDownload"
      >
        Export
      </el-button>
      <el-date-picker
        style="width: 140px"
        width="140px"
        v-model="start"
        class="filter-item"
        type="date"
        placeholder="Dari"
      >
      </el-date-picker>
      <el-date-picker
        style="margin-left: 8px; width: 140px; margin-right: 10px"
        v-model="end"
        class="filter-item"
        type="date"
        placeholder="Sampai"
      >
      </el-date-picker>
      <el-button
        class="filter-item"
        type="primary"
        icon="el-icon-edit"
        @click="handleFilterByDate"
      >
        Filter
      </el-button>
    </div>

    <el-table
      :key="tableKey"
      v-loading="listLoading"
      :data="returnpembelians"
      :default-sort="{ prop: 'id' }"
      border
      fit
      highlight-current-row
      style="width: 100%"
      @sort-change="sortChange"
    >
      <el-table-column
        sortable
        label="ID"
        prop="cashin"
        align="center"
        width="80"
      >
        <template slot-scope="{ row }">
          <span>{{ row.id }}</span>
        </template>
      </el-table-column>
      <el-table-column
        label="Supplier"
        min-width="150px"
        sortable
        prop="name"
      >
        <template slot-scope="{ row }">
          <span>{{
            row.contact != null ? row.contact.name : ""
          }}</span>
        </template>
      </el-table-column>
      <el-table-column
        label="Tagihan"
        width="150px"
        align="center"
        sortable
        prop="total"
      >
        <template slot-scope="{ row }">
          <span>{{ handleCurrency(row.total) }}</span>
        </template>
      </el-table-column>
      <el-table-column
        label="Jumlah dibayar"
        width="150px"
        align="center"
        sortable
        prop="paid"
      >
        <template slot-scope="{ row }">
          <span>{{ handleCurrency(row.paid) }}</span>
        </template>
      </el-table-column>
      <el-table-column
        label="Aksi"
        align="left"
        width="80"
        class-name="small-padding fixed-width"
      >
        <template slot-scope="{ row }">
          <el-popover trigger="hover" placement="top">
            <el-button
              v-if="row.debt > 0"
              type="primary"
              size="mini"
              @click="handleUpdate(row)"
            >
              Bayar
            </el-button>
            <div slot="reference" class="name-wrapper">
              <el-tag size="medium">Aksi</el-tag>
            </div>
            <el-button
              type="danger"
              size="mini"
              @click="handleDelete(row)"
              v-if="checkPermission(['admin', 'finance'])"
            >
              Delete
            </el-button>
            <br />
            <br />
            <router-link style="margin-right:15px;" :to="'/pembelian/detail/' + row.id">
              <el-button size="mini" type="warning">
                Detail
              </el-button>
            </router-link>

            <router-link :to="'/kredit/detail/' + row.id">
              <el-button size="mini" type="warning">
                Histori Pembayaran
              </el-button>
            </router-link>
          </el-popover>
        </template>
      </el-table-column>
      <el-table-column
        label="Staff"
        width="150px"
        align="center"
        sortable
        prop="staff"
      >
        <template slot-scope="{ row }">
          <span>{{ row.staff }}</span>
        </template>
      </el-table-column>

      <el-table-column
        label="Date"
        width="150px"
        align="center"
        sortable
        prop="date"
      >
        <template slot-scope="{ row }">
          <span>{{ row.date }}</span>
        </template>
      </el-table-column>
      <el-table-column label="Kas" width="150px" align="center">
        <template slot-scope="{ row }">
          <span>{{ row.cashin.name }}</span>
        </template>
      </el-table-column>
    </el-table>

    <pagination
      v-show="total > 0"
      :page.sync="listQuery.page"
      :limit.sync="listQuery.limit"
      :current-page.sync="listQuery.page"
      :total="total"
      :page-size="20"
      :page-sizes="[10, 20, 50]"
      @current-change="(val) => listQuery.page = val"
    />

    <el-dialog
      :title="textMap[dialogStatus]"
      :visible.sync="dialogFormVisible"
    >
      <el-form
        label-position="top"
        :inline="true"
        ref="dataForm"
        :rules="rules"
        :model="temp"
        label-width="180px"
        style="width: 100%; margin-left: 50px"
      >
        <el-form-item
          class="k"
          label="Pilih Data Pembelian"
          v-if="dialogStatus == 'create'"
        >
          <el-select
            filterable
            v-model="stockout_id"
            required
            class="filter-item"
            placeholder="Pilih"
            @change="stocktransactionDetail()"
          >
            <el-option
              v-for="item in data_stockout"
              :key="item.id"
              :label="item.id"
              :value="item.id"
            />
          </el-select>
        </el-form-item>
        <el-form-item
          class="k"
          label="Supplier"
          v-if="dialogStatus == 'create'"
        >
          <el-select
            filterable
            v-model="contact_id"
            required
            class="filter-item"
            placeholder="Pilih"
            @change="filterProductPrice()"
          >
            <el-option
              v-for="item in kontak"
              :key="item.id"
              :label="item.name"
              :value="item.id"
            />
          </el-select>
        </el-form-item>
        <el-form-item
          class="k"
          label="Tgl Transaksi"
          v-if="dialogStatus == 'create'"
        >
          <el-date-picker
            v-model="dates"
            type="date"
            format="dd-MM-yyyy"
            placeholder="Tanggal Transaksi"
          >
          </el-date-picker>
        </el-form-item>

        <template v-if="dialogStatus === 'create'">
          <div
            v-for="(all, index) in kasIn.all"
            :key="index"
            style="
              width: 100%;
              padding-left: 4px;
              display: flex;
              flex-wrap: wrap;
            "
          >
            <el-form-item class="k" :label="index == 0 ? 'Barang' : ''">
              <el-select
                v-model="all.product_id"
                filterable
                placeholder="Pilih"
                @change="onChangeProduct(all)"
              >
                <el-option
                  v-for="item in product"
                  :key="item.id"
                  :label="item.name"
                  :value="item.id"
                >
                </el-option>
              </el-select>
            </el-form-item>
            <el-form-item
              style="width: 120px"
              class="k"
              :label="index == 0 ? 'Jumlah Barang' : ''"
            >
              <el-input
                v-model="all.qty"
                :value="all.qty"
                required
                type="number"
                min="1"
                placeholder="Jumlah Barang"
                @change="onChangeQty(all)"
              />
            </el-form-item>
            <el-form-item
              style="width: 140px"
              class="k"
              :label="index == 0 ? 'Satuan' : ''"
            >
              <el-select
                v-model="all.multi_unit_id"
                filterable
                disabled
                placeholder="Pilih"
                @change="onChangeMultiUnit(all)"
              >
                <el-option
                  :label="setSatuanDasar(all.product_id)"
                  value="none"
                ></el-option>
                <el-option
                  v-for="item in all.multi_unit"
                  :key="item.id"
                  :label="item.multi_unit.name"
                  :value="item.id"
                >
                </el-option>
              </el-select>
            </el-form-item>
            <el-form-item
              class="k"
              :label="index == 0 ? 'Harga Satuan' : ''"
            >
              <v-money-spinner
                autocomplete="off"
                v-bind="config"
                v-model="all.harga"
                required
                type="text"
                placeholder="Rp 0"
                @change="onChangeQty(all)"
              ></v-money-spinner>
            </el-form-item>
            <el-form-item
              class="k"
              :label="index == 0 ? 'Sub Total' : ''"
            >
              <v-money-spinner
                autocomplete="off"
                v-bind="config"
                disabled
                v-model="all.total"
                type="numeric"
                :min="0.01"
                :step="0.01"
                :max="2500"
                placeholder="Rp 0"
                @change="onChangeTotal()"
              ></v-money-spinner>
            </el-form-item>
            <el-form-item
              class="k"
              :style="index == 0 ? 'margin-top:50px' : ''"
            >
              <el-button
                v-if="index != 0"
                style="height: 30px"
                type="primary"
                @click="deleteFormProdukByIndex(index)"
              >
                Hapus
              </el-button>
            </el-form-item>
          </div>
        </template>

        <el-button
          style="margin: 20px 0"
          type="primary"
          @click="addFind"
          v-if="dialogStatus == 'create'"
        >
          Tambah Produk
        </el-button>
        <br/>

        <el-form-item class="k" label="Masuk Ke Kas">
          <el-select
            v-model="cashin_id"
            required
            class="filter-item"
            placeholder="Pilih"
            @change="onChangeModal($event)"
          >
            <el-option
              v-for="item in kas"
              :key="item.id"
              :label="item.name"
              :value="item.id"
            />
          </el-select>
        </el-form-item>
        <el-form-item class="k" label="Jumlah Pembayaran">
          <v-money-spinner
            autocomplete="off"
            placeholder="Rp 0"
            v-model="jumlah_bayar"
            v-bind="config"
            @change="handleChangeText()"
          ></v-money-spinner>
        </el-form-item>

        <h3 v-if="total_kasIn != ''">
          Total : {{ handleCurrency(total_kasIn) }}
        </h3>
        <h3 v-if="kurang_bayar != ''">
          Kekurangan : {{ handleCurrency(kurang_bayar) }}
        </h3>
        <h3 v-if="sisa_bayar != ''">
          Kembalian : {{ handleCurrency(sisa_bayar) }}
        </h3>
      </el-form>

      <div
        slot="footer"
        class="dialog-footer"
        style="display: flex; flex-wrap: wrap; justify-content: center"
      >
        <el-button
          style="margin: 20px 10px"
          @click="dialogFormVisible = false"
        >
          Cancel
        </el-button>
        <el-button
          style="margin: 20px 10px"
          :loading="loading"
          type="primary"
          :disabled="button < 1"
          @click="
            dialogStatus === 'create' ? createData() : updateData()
          "
        >
          Simpan
        </el-button>
      </div>
    </el-dialog>

    <el-dialog :visible.sync="dialogPvVisible" title="Reading statistics">
      <el-table
        :data="pvData"
        border
        fit
        highlight-current-row
        style="width: 100%"
      >
        <el-table-column prop="key" label="Channel" />
        <el-table-column prop="pv" label="Pv" />
      </el-table>
      <span slot="footer" class="dialog-footer">
        <el-button type="primary" @click="dialogPvVisible = false"
          >Confirm</el-button
        >
      </span>
    </el-dialog>
  </div>
</template>

<script>
import {
  fetchList,
  fetchPv,
  createArticle,
  updateArticle,
} from "@/api/article";
import waves from "@/directive/waves"; // waves directive
import { parseTime } from "@/utils";
import Pagination from "@/components/Pagination"; // secondary package based on el-pagination
import axios from "@/api/axios";
import qs from "qs";
import { mapGetters } from "vuex";
import checkPermission from "@/utils/permission"; // 权限判断函数
import Cookies from "js-cookie";

const calendarTypeOptions = [
  {
    key: "cash",
    display_name: "cash",
  },
  {
    key: "modal",
    display_name: "modal",
  },
];

// arr to obj, such as { CN : "China", US : "USA" }
const calendarTypeKeyValue = calendarTypeOptions.reduce((acc, cur) => {
  acc[cur.key] = cur.display_name;
  return acc;
}, {});

export default {
  name: "ComplexTable",
  components: {
    Pagination,
  },
  directives: {
    waves,
  },
  filters: {
    statusFilter(status) {
      const statusMap = {
        published: "success",
        draft: "info",
        deleted: "danger",
      };
      return statusMap[status];
    },
    typeFilter(type) {
      return calendarTypeKeyValue[type];
    },
  },
  computed: {
    ...mapGetters(["name", "avatar", "roles"]),
    returnpembelians () {
      if (_.isEmpty(this.list)) {
        return [];
      }
      const {
        limit,
        page
      } = this.listQuery

      const filtered = this.list.filter(({ contact }) => {
        return (
          !this.search ||
          contact.name.toLowerCase().includes(this.search.toLowerCase())
        );
      })

      this.total = filtered.length || 1

      const slice = filtered.slice(limit * page - limit, limit * page)
      return slice
    }
  },
  data() {
    return {
      lastKeyCode: null,
      discount: '0',
      id: "",
      data_stockout: [],
      start: "",
      index_before: "",
      end: "",
      sisa_bayar: [],
      kurang_bayar: [],
      names: "",
      jatuh_tempo: "",
      jumlah_bayar: '0',
      dates: "",
      category: "",
      kontak: [],
      kas: [],
      search: "",
      product: [],
      contact_id: "",
      cashin_id: "",
      satuan: "",
      producttype: "",
      button: "",
      jenis_barang: "",
      keterangan: "",
      stockout_id: "",
      selling_price: "",
      purchase_price: "",
      qty: "",
      qty_before: 0,
      unit: "",
      from: "",
      to_item: "",
      total_kasIn: "",
      pemasukan: "",
      config: {
        spinner: false,
        step: 10,
        prefix: "Rp ",
        precision: 0,
        decimal: ",",
        thousands: ".",
        bootstrap: true,
        amend: false,
        masked: false,
        allowBlank: true,
      },
      kasIn: {
        all: [
          {
            product_id: "",
            total: '0',
            qty: null,
            harga: '0',
            harga_jual: '0',
            multi_unit: [],
            multi_unit_id: "",
            multi_qty: "",
            selling_price: "",
          },
        ],
      },
      tableKey: 0,
      list: null,
      total: 0,
      listLoading: true,
      listQuery: {
        page: 1,
        limit: 20,
        importance: undefined,
        title: undefined,
        type: undefined,
        sort: "+id",
      },
      importanceOptions: [1, 2, 3],
      calendarTypeOptions,
      cash: [],
      modal: [],
      sortOptions: [
        {
          label: "ID Ascending",
          key: "+id",
        },
        {
          label: "ID Descending",
          key: "-id",
        },
      ],
      statusOptions: ["published", "draft", "deleted"],
      showReviewer: false,
      temp: {
        id: undefined,
        code: "",
        date: "",
        timestamp: new Date(),
        title: "",
        to: "",
        chasin: "",
        total: "",
      },
      dialogFormVisible: false,
      dialogStatus: "",
      textMap: {
        update: "Edit",
        create: "Retur Pembelian",
      },
      dialogPvVisible: false,
      pvData: [],
      rules: {
        type: [
          {
            required: true,
            message: "type is required",
            trigger: "change",
          },
        ],
        // timestamp: [{ type: 'date', required: true, message: 'timestamp is required', trigger: 'change' }],
        title: [
          {
            required: true,
            message: "title is required",
            trigger: "blur",
          },
        ],
      },
      downloadLoading: false,
      loading: false,
    };
  },
  created() {
    this.getStockOut();
    this.getList();
    console.log(this.data_stockout);

    let DD = new Date().getDate();
    let MM = new Date().getMonth() + 1;
    let YYYY = new Date().getFullYear();

    this.dates = `${YYYY}-${MM}-${DD}`;
    this.jatuh_tempo = `${YYYY}-${MM}-${DD}`;
  },
  mounted () {
    // START - script untuk disable Ctrl+J pada browser - terkait muncul halaman downloads ketika scan dengan barcode scanner - ASROFI asrofi
    document.addEventListener('keydown', this.disableKeyDown)
    // END - script untuk disable Ctrl+J pada browser - terkait muncul halaman downloads ketika scan dengan barcode scanner - ASROFI asrofi
  },
  beforeDestroy () {
    document.removeEventListener('keydown', this.disableKeyDown)
  },
  methods: {
    checkPermission,
    disableKeyDown (e) {
      if (this.lastKeyCode === 'Control' && e.keyCode === 74) {
        e.preventDefault()
      }
      this.lastKeyCode = e.key
    },
    getStockOut() {
      axios
        .get("/stock/in")
        .then((res) =>
          console.log(
            (this.data_stockout = res.data.stocktransaction)
          )
        )
        .catch((err) => console.log(err));
    },
    stocktransactionDetail() {
      axios
        .get(`/stock/transaction/detail/${this.stockout_id}`)
        .then((res) => {
          console.log(res.data, "detailllllllllll");
          this.kontak = [res.data.stocktransaction[0]["contact"]];
          this.contact_id =
            res.data.stocktransaction[0]["contact"]["id"];
          const product = res.data.stocktransaction[0][
            "substocktransaction"
          ].map((val) => {
            if (val.product["qty"] >= val.qty) {
              val.product["qty"] = val.qty;
              val.product["purchase_price"] = val.purchase_price;
              return val.product;
            } else {
              return [];
            }
          });
          this.product = product;
          console.log(product, "producttttt");
        })
        .catch((err) => console.log(err));
    },
    handleChangeText(i) {
      if (this.dialogStatus == "create") {
        if (
          this.jumlah_bayar + this.discount > this.total_kasIn ||
          this.jumlah_bayar + this.discount == this.total_kasIn
        ) {
          this.sisa_bayar =
            this.jumlah_bayar + this.discount - this.total_kasIn;
          this.kurang_bayar = "";
        } else {
          this.total_kasIn =
            this.total_kasIn < 1 ? 0 : this.total_kasIn;
          this.jumlah_bayar =
            this.jumlah_bayar < 1 ? 0 : this.jumlah_bayar;
          this.discount = this.discount < 1 ? 0 : this.discount;
          this.kurang_bayar =
            this.total_kasIn - (this.jumlah_bayar + this.discount);

          this.sisa_bayar = "";
        }
      } else {
        this.kurang_bayar =
          this.total_kasIn -
          (this.jumlah_bayar +
            this.Pembayaran_sebelum +
            this.discount);
      }
    },
    getList() {
      this.listLoading = true;
      axios.post("/return/out").then((response) => {
        console.log(response, "responseeeeeeeeeeeeeeeeeeeeeeee");
        this.list = response.data.stocktransaction;
        this.total = response.data.stocktransaction.length;

        // Just to simulate the time of the request
        setTimeout(() => {
          this.listLoading = false;
        }, 1.5 * 1000);
      });

      axios
        .get(`/cashuser?in=` + true, {
          headers: {
            Authorization: "Bearer " + Cookies.get("Admin-Token"),
          },
        })
        .then((response) => {
          this.kas = response.data.cashuser;
        });

      axios.get("/contact/supplier").then((response) => {
        console.log(response);
        this.kontak = response.data.contact;
      });
    },
    handleCurrency(number) {
      const idr = new Intl.NumberFormat("in-IN", {
        style: "currency",
        currency: "IDR",
      }).format(number);
      return idr;
    },

    deleteFormProdukByIndex(i) {
      this.button = 1;
      this.kasIn.all = this.kasIn.all.filter((val, index) => {
        if (i != index) {
          return val;
        }
      });

      this.kasIn.all = this.kasIn.all.filter((val, index) => {
        if (val.qty < 1) {
          this.button = 0;
        }
        return val;
      });
    },

    handleFilter() {
      this.listQuery.page = 1;
      this.getList();
    },
    handleModifyStatus(row, status) {
      this.$message({
        message: "操作Success",
        type: "success",
      });
      row.status = status;
    },
    sortChange(data) {
      const { prop, order } = data;
      if (prop === "id") {
        this.sortByID(order);
      }
    },
    sortByID(order) {
      if (order === "ascending") {
        this.listQuery.sort = "+id";
      } else {
        this.listQuery.sort = "-id";
      }
      this.handleFilter();
    },
    resetTemp() {
      this.temp = {
        id: undefined,
        importance: 1,
        remark: "",
        timestamp: new Date(),
        title: "",
        status: "published",
        type: "",
      };
    },
    handleCreate() {
      this.resetTemp();
      this.dialogStatus = "create";
      this.dialogFormVisible = true;
      this.$nextTick(() => {
        this.$refs["dataForm"].clearValidate();
      });
      this.kasIn.all = [
        {
          product_id: "",
          total: '0',
          qty: "",
          harga: '0',
        },
      ];
      this.contact_id = null
      this.stockout_id = null
      this.cashin_id = null
      this.jumlah_bayar = '0'
      this.total_kasIn = "";
    },
    createData() {
      if (this.total_kasIn > this.jumlah_bayar) {
        this.$notify({
          title: "Gagal",
          message: "Jumlah pembayarn kurang",
          type: "warning",
          duration: 2000,
        });

        return false;
      }
      if (this.sisa_bayar == "") {
        this.sisa_bayar = 0;
      }
      this.jumlah_bayar = this.jumlah_bayar - this.sisa_bayar;

      if (this.jumlah_bayar > 0 && this.cashout_id == "") {
        this.$notify({
          title: "Gagal",
          message: "Anda Harus Memilih Kas",
          type: "warning",
          duration: 2000,
        });

        return false;
      }

      this.loading = true;
      const total = [];
      const qty = [];
      const product_id = [];
      const purchase_price = [];
      this.kasIn.all.map((val, index) => {
        if (val.multi_unit_id && val.multi_unit_id != "none") {
          val.multi_unit.filter((multiVal) => {
            if (val.multi_unit_id == multiVal.id) {
              // alert('sama')
              qty.push(
                parseInt(
                  parseFloat(val.qty)
                    .toFixed(2)
                    .replace(/\d(?=(\d{3})+\.)/g, "$&.")
                ) * parseInt(val.multi_qty)
              );
              purchase_price.push(val.harga / multiVal.multi_qty);
            }
          });
        } else {
          qty.push(val.qty);
          purchase_price.push(parseInt(val.harga));
        }
        total.push(parseInt(val.total));
        product_id.push(val.product_id);
      });
      console.log(product_id);
      if (product_id == []) {
        this.$notify({
          title: "Gagal",
          message: "Anda Harus Menambahkan Barang",
          type: "warning",
          duration: 2000,
        });

        return false;
      }
      let paid = "";

      if (
        this.jumlah_bayar < this.total_kasIn &&
        this.discount > this.total_kasIn
      ) {
        paid = 0;
      }
      if (
        this.jumlah_bayar > this.total_kasIn &&
        this.discount < this.total_kasIn
      ) {
        paid = this.total_kasIn - this.discount;
      }

      if (
        this.jumlah_bayar == this.total_kasIn &&
        this.discount < this.total_kasIn
      ) {
        paid = this.jumlah_bayar - this.discount;
      }

      if (
        this.jumlah_bayar == this.total_kasIn &&
        this.discount == this.total_kasIn
      ) {
        paid = 0;
      }

      if (
        this.jumlah_bayar > this.total_kasIn &&
        this.discount == this.total_kasIn
      ) {
        paid = 0;
      }

      if (
        this.jumlah_bayar < this.total_kasIn &&
        this.discount < this.total_kasIn
      ) {
        if (this.jumlah_bayar + this.discount > this.total_kasIn) {
          paid =
            this.jumlah_bayar -
            (this.jumlah_bayar + this.discount - this.total_kasIn);
        } else {
          paid = this.jumlah_bayar;
        }
      }

      if (
        this.jumlah_bayar > this.total_kasIn &&
        this.discount > this.total_kasIn
      ) {
        paid = 0;
      }
      console.log('pad', paid)
      if (this.jumlah_bayar > 0 && this.cashout_id == "") {
        this.$notify({
          title: "Gagal",
          message: "Anda Harus Memilih Kas",
          type: "warning",
          duration: 2000,
        });

        return false;
      }
      const data = {
        contact_id: this.contact_id,
        cashin_id: this.cashin_id,
        product_id,
        qty,
        total,
        payment_due: this.jatuh_tempo,
        paid: this.jumlah_bayar,
        purchase_price,
        date: this.dates,
        staff: this.name,
      };
      console.log(data);
      const encodedValues = qs.stringify(data, {
        allowDots: true,
      });
      axios
        .post("/return/out/create", encodedValues)
        .then((response) => {
          this.loading = false;

          this.getList();
          this.dialogFormVisible = false;
          this.$notify({
            title: "Success",
            message: "Created Successfully",
            type: "success",
            duration: 2000,
          });
        })
        .catch((err) => {
          this.loading = false;

          this.listLoading = false;
          this.$notify({
            title: "Gagal",
            message: " Anda Belum Melengkapi Data",
            type: "warning",
            duration: 2000,
          });
        });
      // }
      // })
    },
    handleUpdate(row) {
      console.log(row.id);
      this.id = row.id;
      // this.unit = row.unit
      // this.producttype = row.producttype.id == '' ? row.producttype : row.producttype.id
      // this.qty = row.qty

      this.total_kasIn = row.total;
      this.cashin_id = row.cashin_id;
      this.jatuh_tempo = row.payment_due;
      this.total_kasIn = row.total;
      this.kurang_bayar = row.debt;
      this.discount = row.discount;
      this.dialogStatus = "update";
      this.Pembayaran_sebelum = row.paid;
      this.dialogFormVisible = true;
      this.$nextTick(() => {
        this.$refs["dataForm"].clearValidate();
      });
      this.ids = row.id;
      this.names = row.cashout.name;
      this.selling_price = row.selling_price;
      this.purchase_price = row.purchase_price;
      this.unit = row.unit;
      this.qty = row.qty;
      this.dialogStatus = "update";
      this.dialogFormVisible = true;
    },
    updateData() {
      this.listLoading = true;
      this.loading = true;
      const data = {
        cashout_id: this.cashout_id,
        total: this.jumlah_bayar,
        payment_due: this.jatuh_tempo,
      };

      axios
        .put(`/stock/in/paid/${this.ids}`, data)
        .then((response) => {
          this.loading = false;

          this.getList();
          this.dialogFormVisible = false;
          this.$notify({
            title: "Success",
            message: "Update Successfully",
            type: "success",
            duration: 2000,
          });
          throw new Error("Something went badly wrong!");
        })
        .catch((err) => {
          this.loading = false;
          if (err.response.status == 400) {
            this.$notify({
              title: "Gagal",
              message: err.response.data.error,
              type: "warning",
              duration: 2000,
            });
          } else {
            this.$notify({
              title: "Gagal",
              message: "Server Error",
              type: "warning",
              duration: 2000,
            });
          }
        });
    },

    filterProductPrice() {
      axios
        .get(`/product/goods?contact_id=${this.contact_id}`)
        .then((response) => {
          console.log(response.data);
          this.kasIn.all = {};
          this.kasIn.all = [
            {
              product_id: "",
              total: 0,
              qty: "",
              harga: 0,
            },
          ];

          this.product = response.data.product.filter((val) => {
            return val.category != "service"
          });
        });
    },

    handleDelete(row) {
      this.listLoading = true;
      const findIndex = this.list.findIndex(f => f.id === row.id)

      this.$confirm("Apakah anda serius mau menghapus ?", "Warning", {
        confirmButtonText: "OK",
        cancelButtonText: "Cancel",
        type: "warning",
      })
        .then(() => {
          axios
            .delete(`/stock/transaction/delete/${row.id}`)
            .then((response) => {
              this.listLoading = false;

              this.$notify({
                title: "Success",
                message: "Delete Successfully",
                type: "success",
                duration: 2000,
              });
              this.list.splice(findIndex, 1);
            })
            .catch((err) => {
              this.listLoading = false;
              this.$notify({
                title: "Error",
                message: "Server Error",
                type: "warning",
                duration: 2000,
              });
            });
        })
        .catch(() => {
          this.listLoading = false;
          this.$message({
            type: "info",
            message: "Delete canceled",
          });
        });
    },
    handleFetchPv(pv) {
      fetchPv(pv).then((response) => {
        this.pvData = response.data.pvData;
        this.dialogPvVisible = true;
      });
    },
    handleDownload() {
      this.downloadLoading = true;
      import("@/vendor/Export2Excel").then((excel) => {
        const tHeader = [
          "id",
          "supplier",
          "pembayaran",
          "staff",
          "date",
        ];
        const filterVal = [
          "id",
          "name",
          "total",
          "staff",
          "created_at",
        ];
        const data = this.formatJson(filterVal);
        excel.export_json_to_excel({
          header: tHeader,
          data,
          filename: "Retur Pembelian",
        });
        this.downloadLoading = false;
      });
    },
    formatJson(filterVal) {
      return this.list.map((v) =>
        filterVal.map((j) => {
          v["name"] = v.contact.name;
          return v[j];
        })
      );
    },
    getSortClass: function (key) {
      const sort = this.listQuery.sort;
      return sort === `+${key}` ? "ascending" : "descending";
    },
    onChangeCash(event) {
      console.log(event);
    },
    onChangeModal(event) {
      console.log(event);
    },
    addFind() {
      console.log(this.kasIn.all);
      this.button = 1;
      this.kasIn.all.push({
        product_id: "",
        total: '',
        qty: "",
        harga: '',
      });
    },
    deleteFind() {
      this.kasIn.all.pop();
    },
    onChangeTotal() {
      const total = this.kasIn.all.reduce(function (accumulator, item) {
        console.log(item.total);
        return accumulator + parseInt(item.total);
      }, 0);
      this.total_kasIn = total;
    },
    onChangeProduct(item) {
      // ? check produk tidak boleh sama dalam satu transaksi
      if (!_.isEmpty(this.kasIn.all)) {
        const find = this.kasIn.all.filter(f => f.product_id === item.product_id)
        console.log(find)

        if (find && find.length > 1) {
          item.product_id = null
          return this.$notify({
            title: "Tidak boleh !",
            message: "Produk tidak boleh sama dalam 1 transaksi",
            type: "warning",
            duration: 2000,
          });
        }
      }

      const findIndex = this.kasIn.all.findIndex(f => f.id === item.id)

      if (findIndex !== -1) {
        this.index_before = findIndex
      }
      const produk = this.product.find(f => f.id === item.product_id)

      if (produk) {
        this.qty_before = produk.qty
        item.qty = produk.qty
        if (produk.qty > 0) {
          this.button = 1;
        }
        item.harga = produk.purchase_price;
        item.multi_unit_id = 'none'
      }
    },
    onChangeMultiUnit(item) {
      const findIndex = this.kasIn.all.findIndex(f => f.product_id === item.product_id)
      if (findIndex !== -1) {
        this.index_before = findIndex;
      }
      if (item.multi_unit_id == "none") {
        const produk = this.product.find(f => f.id === item.product_id)

        if (produk) {
          this.qty_before = produk.qty
          item.qty = this.qty_before;
          item.harga = produk.purchase_price;
          item.total = item.qty * produk.purchase_price;
        }
      } else {
        item.selling_price = item.harga;
        item.multi_unit.forEach((val) => {
          this.product.forEach((valProd) => {
            if (valProd.id == item.product_id) {
              if (item.multi_unit_id == val.id) {
                const jumlahPerMultiSatuanNow = item.qty * val.multi_qty;
                if (jumlahPerMultiSatuanNow <= this.qty_before) {
                  item.harga = val.multi_qty * valProd.purchase_price;
                  item.multi_qty = val.multi_qty;
                } else {
                  item.multi_unit_id = "none";
                }
              }
            }
          });
        });
      }
    },
    handleFilterByDate() {
      this.listLoading = true;
      let data = {
        start_date: this.start,
        end_date: this.end,
      };
      axios.post(`/return/out`, data).then((response) => {
        this.list = response.data.stocktransaction.map((val) => {
          val["debt"] =
            val.total - val.paid - val.discount < 0
              ? 0
              : val.total - val.paid - val.discount;
          return val;
        });
        this.total = response.data.stocktransaction.length;

        // Just to simulate the time of the request
        setTimeout(() => {
          this.listLoading = false;
        }, 1.5 * 1000);
      });
    },
    onChangeQty(item) {
      if (item) {
        const {
          multi_unit,
          multi_unit_id,
          harga,
          product_id
        } = item

        if (!_.isEmpty(multi_unit)) {
          multi_unit.forEach((val) => {
            this.product.forEach((valProd) => {
              if (valProd.id == product_id) {
                if (multi_unit_id == val.id) {
                  const jumlahPerMultiSatuanNow = item.qty * val.multi_qty;
                  if (jumlahPerMultiSatuanNow <= this.qty_before) {
                    item.harga = val.multi_qty * valProd.purchase_price;
                    item.multi_qty = val.multi_qty;
                  } else {
                    item.multi_unit_id = "none";
                  }
                }
              }
            });
          });
        }
        this.button = item.qty;

        let qty = item.qty
        if (item.qty > this.qty_before) {
          qty = this.qty_before
          item.qty = this.qty_before;
        }

        if (typeof item.qty === 'string') {
          if (_.includes(item.qty, '.')) {
            qty = item.qty.replace(".", "");
          }

          if (_.includes(item.qty, ',')) {
            qty = item.qty.replace(/,/g, ".");
          }
          item.qty = qty;
        }
        const result = qty * +harga;
      
        item.total = result;
        const subtotal = this.kasIn.all.reduce(function (accumulator, item) {
          console.log(item.total);
          return accumulator + parseInt(item.total);
        }, 0);
        this.total_kasIn = subtotal;
      }
    },
    setSatuanDasar (product_id) {
      if (!product_id) {
        return 'Satuan Dasar'
      }

      const product = this.product.find(f => f.id === product_id)

      if (product) {
        return product?.unit?.name || 'Satuan Dasar'
      } else {
        return 'Satuan Dasar'
      }
    }
  },
};
</script>
