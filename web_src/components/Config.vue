<template>
<div class="container-fluid no-padding">
    <!-- <section class="content"> -->
    <div :class="['nav-tabs-custom', 'no-padding', {'col-lg-offset-2':!bShowMore, 'col-lg-8':!bShowMore, 'col-lg-offset-1':bShowMore, 'col-lg-10':bShowMore}]">
        <ul class="nav nav-tabs">
            <li class="active"><a href="#base-config" data-toggle="tab">{{logoText}} 信令服务配置</a></li>
            <li @click.prevent="getSMSList"><a href="#sms-config" data-toggle="tab">{{logoText}} 流媒体服务配置</a></li>
        </ul>
        <div class="tab-content">
            <div class="tab-pane active" id="base-config">
                <form role="form" class="form-horizontal" autocomplete="off" @submit.prevent="onSubmit" data-vv-scope="cms">
                    <div :class="[{'col-md-6': bShowMore, 'col-md-12': !bShowMore}]">
                        <div :class="['form-group' , {'has-error':  errors.has('cms.Serial')}]">
                            <label for="sip-serial" class="control-label" :class="{'col-sm-3':GM&&!bShowMore, 'col-sm-4':!GM||bShowMore}">SIP ID</label>
                            <div :class="{'col-sm-8':GM&&!bShowMore, 'col-sm-7':!GM||bShowMore}">
                                <input id="sip-serial" type="text" class="form-control" name="Serial" data-vv-as="SIP ID" v-validate="'required'" v-model.trim="Serial">
                                <span class="help-block">{{errors.first('Serial')}}</span>
                            </div>
                        </div>
                        <div :class="['form-group' , {'has-error': errors.has('cms.Realm')}]">
                            <label for="sip-realm" class="control-label" :class="{'col-sm-3':GM&&!bShowMore, 'col-sm-4':!GM||bShowMore}">SIP 域</label>
                            <div :class="{'col-sm-8':GM&&!bShowMore, 'col-sm-7':!GM||bShowMore}">
                                <input id="sip-realm" type="text" class="form-control" name="Realm" data-vv-as="SIP 域" v-validate="'required'" v-model.trim="Realm">
                                <span class="help-block">{{errors.first('cms.Realm')}}</span>
                            </div>
                        </div>
                        <div :class="['form-group' , {'has-error': errors.has('cms.Host')}]">
                            <label for="sip-host" class="control-label" :class="{'col-sm-3':GM&&!bShowMore, 'col-sm-4':!GM||bShowMore}">SIP Host</label>
                            <div :class="{'col-sm-8':GM&&!bShowMore, 'col-sm-7':!GM||bShowMore}">
                                <input type="text" id="sip-host" class="form-control" name="Host" data-vv-as="SIP Host" v-validate="'required'" v-model.trim="Host">
                                <span class="help-block">{{errors.first('cms.Host')}}</span>
                            </div>
                        </div>
                        <div :class="['form-group' , {'has-error': errors.has('cms.Port')}]">
                            <label for="sip-port" class="control-label" :class="{'col-sm-3':GM&&!bShowMore, 'col-sm-4':!GM||bShowMore}">SIP 端口</label>
                            <div :class="{'col-sm-8':GM&&!bShowMore, 'col-sm-7':!GM||bShowMore}">
                                <input type="text" id="sip-port" class="form-control" name="Port" data-vv-as="SIP 端口" v-validate="'required|numeric'" v-model.trim="Port">
                                <span class="help-block">{{errors.first('cms.Port')}}</span>
                            </div>
                        </div>
                        <div :class="['form-group' , {'has-error': errors.has('cms.DevicePassword')}]">
                            <label for="sip-dev-pwd" class="control-label" :class="{'col-sm-3':GM&&!bShowMore, 'col-sm-4':!GM||bShowMore}">设备统一接入密码</label>
                            <div :class="{'col-sm-8':GM&&!bShowMore, 'col-sm-7':!GM||bShowMore}">
                                <input type="text" id="sip-dev-pwd" class="form-control" name="DevicePassword" data-vv-as="设备统一接入密码" v-model.trim="DevicePassword">
                                <span class="help-block">{{errors.first('cms.DevicePassword')}}</span>
                            </div>
                        </div>
                        <div :class="['form-group', {'has-error': errors.has('cms.PreferStreamFmt')}]" title="配置直播流优先使用的播放格式">
                            <label for="prefer-stream-fmt" class="control-label" :class="{'col-sm-3':GM&&!bShowMore, 'col-sm-4':!GM||bShowMore}">首选直播格式</label>
                            <div :class="{'col-sm-8':GM&&!bShowMore, 'col-sm-7':!GM||bShowMore}">
                                <select id="prefer-stream-fmt" name="PreferStreamFmt" class="form-control" v-model.trim="PreferStreamFmt">
                                    <option value="">自动选择</option>
                                    <option value="FLV">FLV</option>
                                    <option value="WS_FLV">WS_FLV</option>
                                    <option value="WEBRTC">WEBRTC</option>
                                    <option value="HLS">HLS</option>
                                    <option value="RTMP">RTMP</option>
                                </select>
                                <span class="help-block">{{errors.first('cms.PreferStreamFmt')}}</span>
                            </div>
                        </div>
                        <div :class="{'form-group': true, 'has-error': errors.has('cms.HTTPSPort')}" title="配置为0, 表示不开启HTTPS">
                            <label for="base-https-port" class="control-label" :class="{'col-sm-3':GM&&!bShowMore, 'col-sm-4':!GM||bShowMore}">HTTPS 端口(可选)</label>
                            <div :class="{'col-sm-8':GM&&!bShowMore, 'col-sm-7':!GM||bShowMore}">
                                <input id="base-https-port" type="text" class="form-control" name="HTTPSPort" data-vv-as="HTTPS 端口" v-validate="'numeric'" v-model.trim.number="HTTPSPort" placeholder="默认不开启HTTPS">
                                <span class="help-block">{{errors.first('cms.HTTPSPort')}}</span>
                            </div>
                        </div>
                        <div :class="{'form-group': true, 'has-error': errors.has('cms.HTTPSCertFile')}" title="配置 Cert 证书路径，绝对路径" v-if="HTTPSPort">
                            <label for="base-https-cert-file" class="control-label" :class="{'col-sm-3':GM&&!bShowMore, 'col-sm-4':!GM||bShowMore}">HTTPS Cert 证书路径</label>
                            <div :class="{'col-sm-8':GM&&!bShowMore, 'col-sm-7':!GM||bShowMore}">
                                <input id="base-https-cert-file" type="text" class="form-control" spellcheck="false" autocomplete="off" name="HTTPSCertFile" v-model.trim="HTTPSCertFile" placeholder="配置cert证书绝对路径">
                                <span class="help-block">{{errors.first('cms.HTTPSCertFile')}}</span>
                            </div>
                        </div>
                        <div :class="{'form-group': true, 'has-error': errors.has('cms.HTTPSKeyFile')}" title="配置 Key 证书路径，绝对路径" v-if="HTTPSPort">
                            <label for="base-https-key-file" class="control-label" :class="{'col-sm-3':GM&&!bShowMore, 'col-sm-4':!GM||bShowMore}">HTTPS Key 证书路径</label>
                            <div :class="{'col-sm-8':GM&&!bShowMore, 'col-sm-7':!GM||bShowMore}">
                                <input id="base-https-key-file" type="text" class="form-control" spellcheck="false" autocomplete="off" name="HTTPSKeyFile" v-model.trim="HTTPSKeyFile" placeholder="配置key证书绝对路径">
                                <span class="help-block">{{errors.first('cms.HTTPSKeyFile')}}</span>
                            </div>
                        </div>
                        <div class="form-group">
                            <label class="control-label" :class="{'col-sm-3':GM&&!bShowMore, 'col-sm-4':!GM||bShowMore}">接入控制(可选)</label>
                            <div class="checkbox" :class="{'col-sm-8':GM&&!bShowMore, 'col-sm-7':!GM||bShowMore}">
                                <router-link :to="`/black/1`" title="设备接入黑名单维护">
                                    <i class="fa fa-calendar-times-o" aria-hidden="true"></i> 黑名单&nbsp;&nbsp;
                                </router-link>
                                <router-link :to="`/white/1`" title="设备接入白名单维护">
                                    <i class="fa fa-calendar-check-o" aria-hidden="true"></i> 白名单&nbsp;&nbsp;
                                </router-link>
                                <span class="help-block"></span>
                            </div>
                        </div>
                        <div class="form-group">
                            <label class="control-label" :class="{'col-sm-3':GM&&!bShowMore, 'col-sm-4':!GM||bShowMore}">35114 国密认证</label>
                            <div class="checkbox" :class="{'col-sm-8':GM&&!bShowMore, 'col-sm-7':!GM||bShowMore}">
                                <el-checkbox style="margin-left:-19px;margin-top:-5px;" size="small" v-model.trim="GM" name="GM">
                                    启用国密&nbsp;&nbsp;
                                </el-checkbox>
                                <el-upload action="/api/v1/gm/ca/import" accept=".pem,.cer,.crt,.csr,.der" v-if="GM"
                                    style="display:inline-block;margin-right:7px;" :show-file-list="false"
                                    :on-success="onUploadCACertSuccess" :on-error="onUploadCACertError" :on-progress="onUploadCACertProgress">
                                    <a role="button" title="上传CA国密证书" v-if="!GMCACert">
                                        <i class="fa fa-upload"></i> 上传CA证书
                                    </a>
                                    <a role="button" title="重新上传CA国密证书" v-else>
                                        <i class="fa fa-upload"></i> 重新上传CA证书
                                    </a>
                                </el-upload>
                                <a role="button" title="创建证书请求" @click="$refs['gm-req-dlg'].show()" v-if="GM">
                                    <i class="fa fa-certificate" aria-hidden="true"></i> 创建证书请求
                                </a>
                                <span v-if="GM">&nbsp;&nbsp;</span>
                                <router-link to="/gm/req/1" target="_blank" v-if="GM">
                                    <i class="fa fa-edit"></i> 签发设备证书
                                    <svg xmlns="http://www.w3.org/2000/svg" aria-hidden="true" x="0px" y="0px" viewBox="0 0 100 100" width="15" height="15" class="icon outbound"><path fill="currentColor" d="M18.8,85.1h56l0,0c2.2,0,4-1.8,4-4v-32h-8v28h-48v-48h28v-8h-32l0,0c-2.2,0-4,1.8-4,4v56C14.8,83.3,16.6,85.1,18.8,85.1z"></path> <polygon fill="currentColor" points="45.7,48.7 51.3,54.3 77.2,28.5 77.2,37.2 85.2,37.2 85.2,14.9 62.8,14.9 62.8,22.9 71.5,22.9"></polygon></svg>
                                </router-link>
                                <span class="help-block"></span>
                            </div>
                        </div>
                        <div class="form-group" v-if="GM && GMCerts.length > 0">
                            <label class="control-label" :class="{'col-sm-3':GM&&!bShowMore, 'col-sm-4':!GM||bShowMore}" @click="getGMCerts">35114 国密证书</label>
                            <div :class="{'col-sm-8':GM&&!bShowMore, 'col-sm-7':!GM||bShowMore}">
                                <el-table :data="GMCerts" stripe v-loading="bCertLoading" element-loading-text="加载中...">
                                    <el-table-column min-width="100" label="证书功能" prop="Description" show-overflow-tooltip></el-table-column>
                                    <el-table-column min-width="200" label="签发给" prop="Subject" :formatter="formatCommonName" show-overflow-tooltip></el-table-column>
                                    <el-table-column min-width="200" label="签发者" prop="Issuer" show-overflow-tooltip>
                                        <template slot-scope="scope">
                                            <span v-if="scope.row.Type == 4">
                                                <a role="button" @click="$refs['gm-cert-dlg'].show(scope.row.Name)" title="创建自签名的本地国密证书">
                                                    <i class="fa fa-edit"></i> 自签名
                                                </a>
                                                &nbsp;&nbsp;
                                                <el-upload :action="`/api/v1/gm/cert/import?req=${scope.row.Name}`" accept=".pem,.cer,.crt,.csr,.der"
                                                    style="display:inline-block;margin-right:7px;" :show-file-list="false"
                                                    :on-success="onUploadLocalCertSuccess" :on-error="onUploadLocalCertError" :on-progress="onUploadLocalCertProgress">
                                                    <a role="button" title="上传CA签名的本地国密证书">
                                                        <i class="fa fa-upload"></i> CA签名
                                                    </a>
                                                </el-upload>
                                            </span>
                                            <span v-else>{{formatCommonName(scope.row, scope.column, scope.row.Issuer)}}</span>
                                        </template>
                                    </el-table-column>
                                    <el-table-column min-width="200" label="有效期" :formatter="formatCertDays" show-overflow-tooltip></el-table-column>
                                    <el-table-column min-width="280" label="序列号" prop="SN" show-overflow-tooltip></el-table-column>
                                    <el-table-column min-width="160" label="修改时间" prop="ModAt" show-overflow-tooltip></el-table-column>
                                    <el-table-column min-width="120" label="操作" :fixed="isMobile() ? false : 'right'">
                                        <template slot-scope="scope">
                                            <div class="btn-group">
                                                <a role="button" class="btn btn-info btn-xs" @click.prevent="downloadCert(scope.row.Name)"><i class='fa fa-download'></i> 下载</a>
                                                <a role="button" class="btn btn-danger btn-xs" @click.prevent="removeCert(scope.row.Name, scope.row.Description)"><i class='fa fa-remove'></i> 删除</a>
                                            </div>
                                        </template>
                                    </el-table-column>
                                </el-table>
                            </div>
                        </div>
                        <div class="form-group">
                            <label class="control-label" :class="{'col-sm-3':GM&&!bShowMore, 'col-sm-4':!GM||bShowMore}">其他配置</label>
                            <div class="checkbox" :class="{'col-sm-8':GM&&!bShowMore, 'col-sm-7':!GM||bShowMore}">
                                <el-checkbox title="开启后接口调用会校验登录" style="margin-left:-19px;margin-top:-5px;" size="small" v-model.trim="APIAuth" name="APIAuth">
                                    HTTP 接口鉴权&nbsp;&nbsp;
                                </el-checkbox>
                                <el-checkbox title="开启后流地址访问会校验登录" style="margin-left:-19px;margin-top:-5px;" size="small" v-model.trim="LiveStreamAuth" name="LiveStreamAuth">
                                    流地址鉴权&nbsp;&nbsp;
                                </el-checkbox>
                                <el-checkbox title="开启后不需要事先通过接口拉流, 直播地址直接可用" style="margin-left:-19px;margin-top:-5px;" size="small" v-model.trim="AllowStreamStartByURL" name="AllowStreamStartByURL">
                                    允许直播地址拉流&nbsp;&nbsp;
                                </el-checkbox>
                                <el-checkbox style="margin-left:-19px;margin-top:-5px;" size="small" v-model.trim="SIPLog" name="SIPLog">信令日志&nbsp;&nbsp;</el-checkbox>
                                <p class="text-danger" v-show="remoteAPIAuth === false && !APIAuth">
                                    <br>
                                    <i class="fa fa-exclamation-circle"></i> 接口鉴权关闭状态, 表示不需要登录即可访问系统, 可能导致不可预测的安全风险!
                                    <br>
                                </p>
                                <p class="text-danger" v-show="remoteAPIAuth && !APIAuth">
                                    <br>
                                    <i class="fa fa-exclamation-circle"></i> 关闭接口鉴权, 表示不需要登录即可访问系统, 可能导致不可预测的安全风险, 谨慎操作!
                                    <br>
                                </p>
                                <el-checkbox style="margin-left:-19px;margin-top:-5px;" size="small" v-model.trim="confirmAPIAuth" name="confirmAPIAuth" v-show="remoteAPIAuth && !APIAuth">
                                    <span class="text-danger">我已知悉关闭接口鉴权带来的安全风险</span>
                                </el-checkbox>
                                <span class="help-block"></span>
                            </div>
                        </div>
                        <br v-if="bShowMore"/>
                        <div class="form-group" style="font-weight: bold;padding-top: 15px;padding-bottom: 10px;">
                            <div :class="{'col-sm-offset-3':GM&&!bShowMore, 'col-sm-8':GM&&!bShowMore, 'col-sm-offset-4':!GM||bShowMore, 'col-sm-7':!GM||bShowMore}">
                                <a href="#" class="form-control-static text-primary" @click.prevent="bShowMore = !bShowMore;" >
                                    {{bShowMore ? '&lt;&lt;&nbsp;&nbsp;关闭更多配置':'显示更多配置&nbsp;&nbsp;&gt;&gt;'}}
                                </a>
                            </div>
                        </div>
                    </div>
                    <div class="col-md-6" v-if="bShowMore">
                        <div :class="['form-group' , {'has-error': errors.has('cms.TimeServer')}]">
                            <label for="sip-time-server" class="col-sm-4 control-label">校时源(可选)</label>
                            <div class="col-sm-7">
                                <input type="text" id="sip-time-server" class="form-control" name="TimeServer" data-vv-as="校时源" v-validate="" v-model.trim="TimeServer" placeholder="上级国标编号/NTP">
                                <span class="help-block">{{errors.first('cms.TimeServer')}}</span>
                            </div>
                        </div>
                        <div class="form-group">
                            <label for="black-ip-list" class="col-sm-4 control-label">黑名单 IP(可选)</label>
                            <div class="col-sm-7">
                                <input type="text" id="black-ip-list" class="form-control" name="BlackIPList" data-vv-as="黑名单 IP" v-model.trim="BlackIPList" placeholder="选填">
                                <span class="help-block"></span>
                            </div>
                        </div>
                        <div class="form-group">
                            <label for="black-ua-list" class="col-sm-4 control-label">黑名单 UA(可选)</label>
                            <div class="col-sm-7">
                                <input type="text" id="black-ua-list" class="form-control" name="BlackUAList" data-vv-as="黑名单 UA" v-model.trim="BlackUAList" placeholder="选填">
                                <span class="help-block"></span>
                            </div>
                        </div>
                        <div class="form-group">
                            <label class="col-sm-4 control-label">全局过滤通道类型</label>
                            <div class="col-sm-7">
                                <el-select style="width:100%;" size="medium" v-model.trim="dropChannelTypes" multiple filterable allow-create default-first-option placeholder="选填, 需要丢弃的通道类型">
                                    <el-option v-for="(item, idx) in innerChannelTypes" :key="idx" :label="`${idx} - ${item}`" :value="idx">
                                    </el-option>
                                </el-select>
                            </div>
                        </div>
                        <div class="form-group" title="配置新接入设备的默认流传输模式">
                            <label class="col-sm-4 control-label">默认流传输模式</label>
                            <div class="col-sm-7">
                                <el-radio-group v-model.trim="MediaTransport" size="mini" @change="onMediaTransportChange">
                                    <el-radio-button label="TCP"></el-radio-button>
                                    <el-radio-button label="UDP"></el-radio-button>
                                </el-radio-group>
                                &nbsp;&nbsp;
                                <el-radio-group v-model.trim="MediaTransportMode" size="mini" v-show="MediaTransport == 'TCP'">
                                    <el-radio-button label="passive">被动</el-radio-button>
                                    <el-radio-button label="active">主动</el-radio-button>
                                </el-radio-group>
                            </div>
                        </div>
                        <div class="form-group">
                            <label class="col-sm-4 control-label">全局订阅项目</label>
                            <div class="col-sm-7 checkbox">
                                <el-checkbox style="margin-left:-19px;margin-top:-5px;" size="small" v-model.trim="global_catalog_subscribe" name="global_catalog_subscribe">
                                    目录订阅&nbsp;&nbsp;
                                </el-checkbox>
                                <el-checkbox style="margin-left:-19px;margin-top:-5px;" size="small" v-model.trim="global_alarm_subscribe" name="global_alarm_subscribe">
                                    报警订阅&nbsp;&nbsp;
                                </el-checkbox>
                                <el-checkbox style="margin-left:-19px;margin-top:-5px;" size="small" v-model.trim="global_position_subscribe" name="global_position_subscribe">
                                    位置订阅
                                </el-checkbox>
                                <span class="help-block"></span>
                            </div>
                        </div>
                        <div class="form-group">
                            <label class="col-sm-4 control-label">全局通道开启音频</label>
                            <div class="col-sm-7 checkbox">
                                 <el-switch  v-model.trim="GlobalChannelAudio"></el-switch>
                                 <span class="help-block"></span>
                            </div>
                        </div>
                        <div class="form-group">
                            <label class="col-sm-4 control-label">全局通道开启分享</label>
                            <div class="col-sm-7 checkbox">
                                 <el-switch  v-model.trim="GlobalChannelShared"></el-switch>
                                 <span class="help-block"></span>
                            </div>
                        </div>
                        <div class="form-group" v-show="serverInfo.VersionType != '标准版'">
                            <label class="col-sm-4 control-label">电子地图</label>
                            <div class="col-sm-7 checkbox">
                                 <el-switch  v-model.trim="MapEnable"></el-switch>
                                 <span class="help-block"></span>
                            </div>
                        </div>
                        <div :class="['form-group' , {'has-error': errors.has('cms.MapCenter')}]" v-show="serverInfo.VersionType != '标准版' && MapEnable">
                            <label class="col-sm-4 control-label">地图中心(经,纬度)</label>
                            <div class="col-sm-7">
                                <input type="text" class="form-control" name="MapCenter" data-vv-as="地图中心" v-model.trim="MapCenter" placeholder="经度,纬度">
                                <span class="help-block">{{errors.first('cms.MapCenter')}}</span>
                            </div>
                        </div>
                        <div class="form-group" v-show="serverInfo.VersionType != '标准版' && MapEnable">
                            <label class="col-sm-4 control-label">地图包上传(zip)</label>
                            <div class="col-sm-7">
                                <el-upload drag action="/api/v1/map/import" accept=".zip"
                                    :show-file-list="false"
                                    :on-success="onUploadMapSuccess" :on-error="onUploadMapError" :on-progress="onUploadMapProgress"
                                    v-loading.fullscreen.lock="bMapUploading" element-loading-text="地图包上传中..."
                                    ref="map-uploader" id="map-uploader">
                                    <i class="el-icon-upload"></i>
                                    <div class="el-upload__text">将地图包拖到此处，或 <em>点击上传</em><br></div>
                                </el-upload>
                            </div>
                        </div>
                    </div>
                    <div class="col-md-12">
                        <div class="form-group">
                            <div :class="[{'col-sm-offset-5': bShowMore, 'col-sm-offset-4': !GM&&!bShowMore, 'col-sm-offset-3': GM&&!bShowMore, 'col-sm-7': true}]">
                                <button type="submit" class="btn btn-primary" :disabled="isBasicNoChange || errors.any('cms') || (remoteAPIAuth && !APIAuth && !confirmAPIAuth) || bCommitting || bUploading">保存</button>
                            </div>
                        </div>
                    </div>
                    <div class="clearfix"></div>
                </form>
            </div>
            <div class="tab-pane" id="sms-config">
                <form role="form" class="form-horizontal" autocomplete="off" @submit.prevent="onSubmitSMS" data-vv-scope="sms">
                    <div class="form-group" v-if="smss.length <= 0">
                        <div class="col-sm-12">
                            <div class="alert text-center no-margin">SMS {{smstip}}</div>
                        </div>
                    </div>
                    <div :class="[{'col-md-6': bShowMore, 'col-md-12': !bShowMore}]">
                        <div class="form-group" v-if="smss.length > 0">
                            <label class="col-sm-4 control-label">SMS 服务</label>
                            <div class="col-sm-7">
                                <select class="form-control" v-model.trim="smsserial" @change="smschange">
                                    <option v-for="(c, idx) in smss" :value="c.Serial" :key="idx">{{c.Serial}}</option>
                                </select>
                            </div>
                        </div>
                        <div v-if="smsbaseconfig.Host != undefined && smss.length > 0">
                            <div :class="['form-group' , {'has-error': errors.has('sms.smsHost')}]" title="内部通信收流. 启用外网IP收流后, 此处配置信令服务可访问的局域网IP如：127.0.0.1">
                                <label class="col-sm-4 control-label">本地|内网 IP</label>
                                <div class="col-sm-7">
                                    <input type="text" class="form-control" placeholder="内部通信收流. 启用外网IP收流后, 此处配置信令服务可访问的局域网IP如：127.0.0.1" name="smsHost" data-vv-as="本地|内网 IP" v-validate="'required'" v-model.trim="smsbaseconfig.Host">
                                    <span class="help-block">{{errors.first('sms.smsHost')}}</span>
                                </div>
                            </div>
                            <div :class="['form-group' , {'has-error': errors.has('sms.WanIP')}]">
                                <label class="col-sm-4 control-label">外网 IP(可选)</label>
                                <div class="col-sm-7">
                                    <input type="text" class="form-control" name="WanIP" data-vv-as="外网 IP" v-model.trim="smsbaseconfig.WanIP" placeholder="选填">
                                    <span class="help-block">{{errors.first('sms.WanIP')}}</span>
                                </div>
                            </div>
                            <div :class="['form-group' , {'has-error': errors.has('sms.RecordDir')}]">
                                <label class="col-sm-4 control-label">云录像目录</label>
                                <div class="col-sm-7">
                                    <input type="text" class="form-control" name="RecordDir" data-vv-as="云录像目录" v-model.trim="smsbaseconfig.RecordDir">
                                    <span class="help-block">{{errors.first('sms.RecordDir')}}</span>
                                </div>
                            </div>
                            <div :class="['form-group', {'has-error': errors.has('sms.RTSPPort')}]">
                                <label class="col-sm-4 control-label">RTSP 端口(可选)</label>
                                <div class="col-sm-7">
                                    <input type="text" class="form-control" name="RTSPPort" data-vv-as="RTSP 端口" v-validate="'numeric'" v-model.trim.number="smsbaseconfig.RTSPPort" placeholder="选填">
                                    <span class="help-block">{{errors.first('sms.RTSPPort')}}</span>
                                </div>
                            </div>
                            <div :class="['form-group' , {'has-error': errors.has('sms.RTMPPort')}]">
                                <label class="col-sm-4 control-label">RTMP 端口</label>
                                <div class="col-sm-7">
                                    <input type="text" class="form-control" name="RTMPPort" data-vv-as="RTMP 端口" v-validate="'required|numeric'" v-model.trim.number="smsbaseconfig.RTMPPort">
                                    <span class="help-block">{{errors.first('sms.RTMPPort')}}</span>
                                </div>
                            </div>
                            <div class="form-group">
                                <label class="col-sm-4 control-label">其他配置</label>
                                <div class="col-sm-7 checkbox">
                                    <el-checkbox title="加快启播速度,相应也会增大一点延时" style="margin-left:-19px;margin-top:-5px;" size="small" v-model.trim="smsbaseconfig.GOPCache" name="GOPCache">直播秒开</el-checkbox>
                                    <br v-if="bShowMore"/><br v-if="bShowMore"/><span v-if="!bShowMore">&nbsp;&nbsp;&nbsp;&nbsp;</span>
                                    <el-checkbox style="margin-left:-19px;margin-top:-5px;" size="small" v-model.trim="smsbaseconfig.SIPLog" name="SIPLog">信令日志</el-checkbox>
                                    <br v-if="bShowMore"/><br v-if="bShowMore"/><span v-if="!bShowMore">&nbsp;&nbsp;&nbsp;&nbsp;</span>
                                    <el-checkbox style="margin-left:-19px;margin-top:-5px;" size="small" v-model.trim="smsbaseconfig.UseWanIPRecvStream" name="UseWanIPRecvStream">外网 IP 收流</el-checkbox>
                                    <br v-if="bShowMore"/><br v-if="bShowMore"/><span v-if="!bShowMore">&nbsp;&nbsp;&nbsp;&nbsp;</span>
                                    <el-checkbox title="直接输出H265_FLV，不做转码，防止服务器性能过载" style="margin-left:-19px;margin-top:-5px;" size="small" v-model.trim="smsbaseconfig.OutHevc" name="OutHevc">H265_FLV</el-checkbox>
                                </div>
                            </div>
                            <br v-if="bShowMore"/>
                            <div class="form-group" style="font-weight: bold;padding-top: 15px;padding-bottom: 10px;">
                                <div class="col-sm-offset-4 col-sm-7">
                                    <a href="#" class="form-control-static text-primary" @click.prevent="bShowMore = !bShowMore;" >
                                        {{bShowMore ? '&lt;&lt;&nbsp;&nbsp;关闭更多配置':'显示更多配置&nbsp;&nbsp;&gt;&gt;'}}
                                    </a>
                                </div>
                            </div>
                        </div>
                    </div>
                    <div class="col-md-6" v-if="bShowMore && smsbaseconfig.Host != undefined && smss.length > 0">
                        <div :class="['form-group' , {'has-error': errors.has('sms.smsSerial')}]" title="内部通信使用">
                            <label class="col-sm-4 control-label">SMS ID</label>
                            <div class="col-sm-7">
                                <input type="text" class="form-control" name="smsSerial" data-vv-as="SMS ID" v-validate="'required'" v-model.trim="smsbaseconfig.Serial">
                                <span class="help-block">{{errors.first('sms.smsSerial')}}</span>
                            </div>
                        </div>
                        <div :class="['form-group' , {'has-error': errors.has('sms.smsRealm')}]" title="内部通信使用">
                            <label class="col-sm-4 control-label">SMS 域</label>
                            <div class="col-sm-7">
                                <input type="text" class="form-control" name="smsRealm" data-vv-as="SMS 域" v-validate="'required'" v-model.trim="smsbaseconfig.Realm">
                                <span class="help-block">{{errors.first('sms.smsRealm')}}</span>
                            </div>
                        </div>
                        <div :class="['form-group' , {'has-error': errors.has('sms.smsPort')}]" title="内部通信使用">
                            <label class="col-sm-4 control-label">SMS 端口</label>
                            <div class="col-sm-7">
                                <input type="text" class="form-control" name="smsPort" data-vv-as="SMS 端口" v-validate="'required|numeric'" v-model.trim.number="smsbaseconfig.Port">
                                <span class="help-block">{{errors.first('sms.smsPort')}}</span>
                            </div>
                        </div>
                        <div :class="['form-group' , {'has-error': errors.has('sms.GroupID')}]">
                            <label class="col-sm-4 control-label">分组 ID(可选)</label>
                            <div class="col-sm-7">
                                <input type="text" class="form-control" name="GroupID" data-vv-as="分组 ID" v-model.trim="smsbaseconfig.GroupID" placeholder="选填">
                                <span class="help-block">{{errors.first('sms.GroupID')}}</span>
                            </div>
                        </div>
                        <div :class="['form-group' , {'has-error': errors.has('sms.CleanOverDays')}]">
                            <label class="col-sm-4 control-label">录像保留(天数)</label>
                            <div class="col-sm-7">
                                <input type="number" title="默认不配置会永久保存，直到触发下面的清理阀值" class="form-control" name="CleanOverDays" data-vv-as="录像保留(天数)" v-model.trim.number="smsbaseconfig.CleanOverDays" placeholder="选填">
                                <span class="help-block">{{errors.first('sms.CleanOverDays')}}</span>
                            </div>
                        </div>
                        <div :class="['form-group' , {'has-error': errors.has('sms.CleanFreespacePercent')}]">
                            <label class="col-sm-4 control-label">存储清理阀值(%)</label>
                            <div class="col-sm-7">
                                <input type="number" class="form-control" name="CleanFreespacePercent" data-vv-as="存储清理阀值(%)" v-model.trim.number="smsbaseconfig.CleanFreespacePercent" placeholder="选填">
                                <span class="help-block">{{errors.first('sms.CleanFreespacePercent')}}</span>
                            </div>
                        </div>
                        <div :class="['form-group' , {'has-error': errors.has('sms.CleanFreespaceSize')}]">
                            <label class="col-sm-4 control-label">存储清理阀值(MB)</label>
                            <div class="col-sm-7">
                                <input type="number" class="form-control" name="CleanFreespaceSize" data-vv-as="存储清理阀值(MB)" v-model.trim.number="smsbaseconfig.CleanFreespaceSize" placeholder="选填">
                                <span class="help-block">{{errors.first('sms.CleanFreespaceSize')}}</span>
                            </div>
                        </div>
                        <div :class="['form-group' , {'has-error': errors.has('sms.TCPPortRange')}]">
                            <label class="col-sm-4 control-label">收流端口区间(TCP)</label>
                            <div class="col-sm-7">
                                <input type="text" class="form-control" name="TCPPortRange" data-vv-as="收流TCP端口区间" v-model.trim="smsbaseconfig.TCPPortRange" placeholder="选填">
                                <span class="help-block">{{errors.first('sms.TCPPortRange')}}</span>
                            </div>
                        </div>
                        <div :class="['form-group' , {'has-error': errors.has('sms.UDPPortRange')}]">
                            <label class="col-sm-4 control-label">收流端口区间(UDP)</label>
                            <div class="col-sm-7">
                                <input type="text" class="form-control" name="UDPPortRange" data-vv-as="收流UDP端口区间" v-model.trim="smsbaseconfig.UDPPortRange" placeholder="选填">
                                <span class="help-block">{{errors.first('sms.UDPPortRange')}}</span>
                            </div>
                        </div>
                        <div :class="['form-group' , {'has-error': errors.has('sms.RTCPortRange')}]">
                            <label class="col-sm-4 control-label">WebRTC端口区间(UDP)</label>
                            <div class="col-sm-7">
                                <input type="text" class="form-control" name="RTCPortRange" data-vv-as="WebRTC播放端口区间(UDP)" v-model.trim="smsbaseconfig.RTCPortRange" placeholder="选填">
                                <span class="help-block">{{errors.first('sms.RTCPortRange')}}</span>
                            </div>
                        </div>
                    </div>
                    <div class="col-md-12" v-if="smss.length > 0">
                        <div class="form-group">
                            <div :class="[{'col-sm-offset-5': bShowMore, 'col-sm-offset-4': !bShowMore,'col-sm-7': true}]">
                                <button type="submit" class="btn btn-primary" :disabled="isSMSNoChange || errors.any('sms') || bCommitting || bUploading">保存</button>
                            </div>
                        </div>
                    </div>
                    <div class="clearfix"></div>
                </form>
            </div>
        </div> <!-- /.tab-content -->
    </div>
    <!-- </section> -->
    <GmReqDlg ref="gm-req-dlg" @submit="getGMCerts"></GmReqDlg>
    <GmCertDlg ref="gm-cert-dlg" @submit="getGMCerts"></GmCertDlg>
</div>
</template>

<script>
import GmReqDlg from 'components/GmReqDlg.vue';
import GmCertDlg from 'components/GmCertDlg.vue';
import $ from "jquery";
import moment from 'moment';
import { mapState, mapActions } from "vuex";

export default {
    data() {
        return {
            bCommitting: false,
            bUploading: false,
            bCertLoading: false,
            bMapUploading: false,
            Serial: "",
            Realm: "",
            Host: "",
            Port: 0,
            DevicePassword: "",
            TimeServer: "",
            PreferStreamFmt: "",
            AckTimeout: 0,
            KeepaliveTimeout: 0,
            APIAuth: false,
            remoteAPIAuth: undefined,
            confirmAPIAuth: false,
            LiveStreamAuth: false,
            SIPLog: false,
            AllowStreamStartByURL: false,
            MapEnable: false,
            MapCenter: "",
            BlackIPList: "",
            BlackUAList: "",
            remoteBasicData: "",
            remoteSMSData: "",
            smsserial: "",
            smss: [],
            sms: {},
            smsbaseconfig: {},
            smstip: "流媒体服务尚未启动",
            HTTPSPort: "",
            HTTPSCertFile: "",
            HTTPSKeyFile: "",
            timer: 0,
            bShowMore: false,
            GlobalChannelShared: false,
            GlobalChannelAudio: false,
            GlobalDeviceCatalogSubscribeInterval: 0,
            GlobalDeviceAlarmSubscribeInterval: 0,
            GlobalDevicePositionSubscribeInterval: 0,
            globalDeviceCatalogSubscribeInterval: 0, //backup
            globalDeviceAlarmSubscribeInterval: 0, //backup
            globalDevicePositionSubscribeInterval: 0, //backup
            global_catalog_subscribe: false,
            global_alarm_subscribe: false,
            global_position_subscribe: false,
            DropChannelType: "",
            dropChannelTypes: [],
            innerChannelTypes: {
                "134": "报警输入",
                "135": "报警输出",
                "136": "语音输入",
                "137": "语音输出",
                "200": "中心信令",
                "215": "业务分组",
                "216": "虚拟组织",
            },
            GM: false,
            GMCACert: "",
            GMCerts: [],
            MediaTransport: "UDP",
            MediaTransportMode: "passive",
        };
    },
    components: {
        GmReqDlg, GmCertDlg
    },
    watch: {
        APIAuth: function(val, oldVal) {
            if(!val) {
                this.confirmAPIAuth = false;
            }
        }
    },
    mounted() {
        this.getBaseConfig();
        this.getGMCerts();
    },
    beforeDestroy() {
        if (this.timer) {
            clearTimeout(this.timer);
            this.timer = 0;
        }
    },
    methods: {
        ...mapActions([ "getServerInfo" ]),
        async onSubmit() {
            var ok = await this.$validator.validateAll();
            if (!ok) {
                var e = this.errors.items[0];
                this.$message({
                    type: "error",
                    message: e.msg
                });
                $(`[name=${e.field}]`).focus();
                return;
            }
            var commitObject = this.getBasicCommitObject();
            if(this.remoteAPIAuth && !commitObject.APIAuth && !this.confirmAPIAuth) {
                this.$message({
                    type: "error",
                    message: "关闭接口鉴权, 需要确认知悉安全风险"
                });
                return;
            }
            this.bCommitting = true;
            $.get("/api/v1/setbaseconfig", commitObject).then(data => {
                this.$message({
                    type: "success",
                    message: "配置成功！"
                });
                this.getBaseConfig();
                this.getServerInfo();
            }).always(() => {
                this.bCommitting = false;
            })
        },
        async onSubmitSMS() {
            var ok = await this.$validator.validateAll();
            if (!ok) {
                var e = this.errors.items[0];
                this.$message({
                    type: "error",
                    message: e.msg
                });
                $(`[name=${e.field}]`).focus();
                return;
            }
            if (this.smsbaseconfig["CleanOverDays"] && this.smsbaseconfig["CleanOverDays"] < 0) {
                 this.$message({
                    type: "error",
                    message: "录像保留(天数)不能小于0"
                });
                return
            }
            if (this.smsbaseconfig["CleanFreespacePercent"] < 0) {
                 this.$message({
                    type: "error",
                    message: "存储清理阀值(百分比)不能小于0"
                });
                return
            }
            if (this.smsbaseconfig["CleanFreespaceSize"] < 0) {
                 this.$message({
                    type: "error",
                    message: "存储清理阀值(MB))不能小于0"
                });
                return
            }
            this.bCommitting = true;
            $.get("/api/v1/sms/setbaseconfig", this.smsbaseconfig).then(data => {
                this.$message({
                    type: "success",
                    message: "配置保存中,请稍后..."
                });
                this.smstip = "配置保存中,请稍后...";
                this.smsserial = "";
                this.getSMSList();
            }).always(() => {
                this.bCommitting = false;
            })
        },
        getBasicCommitObject() {
            this.DropChannelType = this.dropChannelTypes.join(",");
            if(this.global_catalog_subscribe && this.GlobalDeviceCatalogSubscribeInterval <= 0) {
                if(this.globalDeviceCatalogSubscribeInterval > 0) {
                    this.GlobalDeviceCatalogSubscribeInterval = this.globalDeviceCatalogSubscribeInterval;
                } else {
                    this.GlobalDeviceCatalogSubscribeInterval = 3600;
                }
            } else if(!this.global_catalog_subscribe && this.GlobalDeviceCatalogSubscribeInterval > 0) {
                this.GlobalDeviceCatalogSubscribeInterval = 0;
            }
            if(this.global_alarm_subscribe && this.GlobalDeviceAlarmSubscribeInterval <= 0) {
                if(this.globalDeviceAlarmSubscribeInterval > 0) {
                    this.GlobalDeviceAlarmSubscribeInterval = this.globalDeviceAlarmSubscribeInterval;
                } else {
                    this.GlobalDeviceAlarmSubscribeInterval = 3600;
                }
            } else if(!this.global_alarm_subscribe && this.GlobalDeviceAlarmSubscribeInterval > 0) {
                this.GlobalDeviceAlarmSubscribeInterval = 0;
            }
            if(this.global_position_subscribe && this.GlobalDevicePositionSubscribeInterval <= 0) {
                if(this.globalDevicePositionSubscribeInterval > 0) {
                    this.GlobalDevicePositionSubscribeInterval = this.globalDevicePositionSubscribeInterval;
                } else {
                    this.GlobalDevicePositionSubscribeInterval = 3600;
                }
            } else if(!this.global_position_subscribe && this.GlobalDevicePositionSubscribeInterval > 0) {
                this.GlobalDevicePositionSubscribeInterval = 0;
            }
            return {
                Serial: this.Serial,
                Realm: this.Realm,
                Host: this.Host,
                Port: this.Port,
                DevicePassword: this.DevicePassword,
                TimeServer: this.TimeServer,
                PreferStreamFmt: this.PreferStreamFmt,
                AckTimeout: this.AckTimeout,
                KeepaliveTimeout: this.KeepaliveTimeout,
                APIAuth: this.APIAuth,
                LiveStreamAuth: this.LiveStreamAuth,
                GM: this.GM,
                SIPLog: this.SIPLog,
                AllowStreamStartByURL: this.AllowStreamStartByURL,
                BlackIPList: this.BlackIPList,
                BlackUAList: this.BlackUAList,
                HTTPSPort: this.HTTPSPort,
                HTTPSCertFile: this.HTTPSCertFile,
                HTTPSKeyFile: this.HTTPSKeyFile,
                GlobalChannelShared: this.GlobalChannelShared,
                GlobalChannelAudio: this.GlobalChannelAudio,
                GlobalDeviceCatalogSubscribeInterval: this.GlobalDeviceCatalogSubscribeInterval,
                GlobalDeviceAlarmSubscribeInterval: this.GlobalDeviceAlarmSubscribeInterval,
                GlobalDevicePositionSubscribeInterval: this.GlobalDevicePositionSubscribeInterval,
                DropChannelType: this.DropChannelType,
                MapEnable: this.MapEnable,
                MapCenter: this.MapCenter,
                MediaTransport: this.MediaTransport,
                MediaTransportMode: this.MediaTransportMode,
            };
        },
        getBaseConfig() {
            $.get("/api/v1/getbaseconfig").then(ret => {
                this.Serial = ret.Serial;
                this.Realm = ret.Realm;
                this.Host = ret.Host;
                this.Port = ret.Port;
                this.DevicePassword = ret.DevicePassword;
                this.TimeServer = ret.TimeServer;
                this.PreferStreamFmt = ret.PreferStreamFmt;
                this.AckTimeout = ret.AckTimeout;
                this.KeepaliveTimeout = ret.KeepaliveTimeout;
                this.APIAuth = ret.APIAuth;
                this.remoteAPIAuth = ret.APIAuth;
                this.LiveStreamAuth = ret.LiveStreamAuth || false;
                this.SIPLog = ret.SIPLog;
                this.AllowStreamStartByURL = ret.AllowStreamStartByURL;
                this.BlackIPList = ret.BlackIPList;
                this.BlackUAList = ret.BlackUAList;
                this.HTTPSPort = ret.HTTPSPort || "";
                this.HTTPSCertFile = ret.HTTPSCertFile;
                this.HTTPSKeyFile = ret.HTTPSKeyFile;
                this.GlobalChannelShared = ret.GlobalChannelShared || false;
                this.GlobalChannelAudio = ret.GlobalChannelAudio || false;
                this.GlobalDeviceCatalogSubscribeInterval = ret.GlobalDeviceCatalogSubscribeInterval || 0;
                this.GlobalDeviceAlarmSubscribeInterval = ret.GlobalDeviceAlarmSubscribeInterval || 0;
                this.GlobalDevicePositionSubscribeInterval = ret.GlobalDevicePositionSubscribeInterval || 0;

                this.globalDeviceCatalogSubscribeInterval = this.GlobalDeviceCatalogSubscribeInterval;
                this.globalDeviceAlarmSubscribeInterval = this.GlobalDeviceAlarmSubscribeInterval;
                this.globalDevicePositionSubscribeInterval = this.GlobalDevicePositionSubscribeInterval;

                this.global_catalog_subscribe = this.GlobalDeviceCatalogSubscribeInterval > 0;
                this.global_alarm_subscribe = this.GlobalDeviceAlarmSubscribeInterval > 0;
                this.global_position_subscribe = this.GlobalDevicePositionSubscribeInterval > 0;
                this.DropChannelType = ret.DropChannelType || "";
                if(this.DropChannelType) {
                    this.dropChannelTypes = this.DropChannelType.split(",");
                } else {
                    this.dropChannelTypes = [];
                }
                this.GM = !!ret.GM;
                this.GMCACert = ret.GMCACert || "";
                this.MapEnable = ret.MapEnable || false;
                this.MapCenter = ret.MapCenter || "";
                this.MediaTransport = ret.MediaTransport || "UDP";
                this.MediaTransportMode = ret.MediaTransportMode || "passive";

                this.remoteBasicData = JSON.stringify(this.getBasicCommitObject());
            });
        },
        onUploadCACertSuccess(res, file, fileList) {
            this.bUploading = false;
            this.GMCACert = res;
            this.getGMCerts();
            this.$message({
                type: "success",
                message: "上传CA证书成功！"
            });
        },
        onUploadCACertProgress(evt, file, fileList) {
            this.bUploading = true;
        },
        onUploadCACertError(err, file, fileList) {
            this.bUploading = false;
            if (err) {
                this.$message({
                    type: "error",
                    message: err + ""
                })
            }
        },
        onUploadLocalCertSuccess(res, file, fileList) {
            this.bUploading = false;
            this.getGMCerts();
            this.$message({
                type: "success",
                message: "上传本地证书成功！"
            });
        },
        onUploadLocalCertProgress(evt, file, fileList) {
            this.bUploading = true;
        },
        onUploadLocalCertError(err, file, fileList) {
            this.bUploading = false;
            if (err) {
                this.$message({
                    type: "error",
                    message: err + ""
                })
            }
        },
        onUploadMapSuccess(res, file, fileList) {
            this.bMapUploading = false;
            if(res.center && res.center.length > 1) {
                this.MapCenter = `${parseFloat(res.center[0].toFixed(5))},${parseFloat(res.center[1].toFixed(5))}`;
            }
            this.$message({
                type: "success",
                message: "上传地图包成功！"
            });
        },
        onUploadMapProgress(evt, file, fileList) {
            this.bMapUploading = true;
        },
        onUploadMapError(err, file, fileList) {
            this.bMapUploading = false;
            if (err) {
                this.$message({
                    type: "error",
                    message: err + ""
                })
            }
        },
        downloadCert(fname) {
            var link = "/api/v1/gm/cert/download";
            if(fname) {
                link += `?fname=${fname}`;
            }
            window.open(link, "_blank");
        },
        removeCert(fname, fdesc) {
            var msg = "确认删除证书?";
            if(fdesc) {
                msg = `确认删除${fdesc}?`;
            }
            this.$confirm(msg, '提示', {
                confirmButtonText: '确定',
                cancelButtonText: '取消',
                type: 'warning'
            }).then(() => {
                var link = "/api/v1/gm/cert/remove";
                if(fname) {
                    link += `?fname=${fname}`;
                }
                $.get(link).then(() => {
                    if(fname == this.GMCACert) {
                        this.GMCACert = "";
                    }
                    this.getGMCerts();
                });
            }).catch(() => {});
        },
        getGMCerts() {
            this.bCertLoading = true;
            $.get("/api/v1/gm/cert/list").then(data => {
                this.GMCerts = data.CertList || [];
            }).always(() => {
                this.bCertLoading = false;
            })
        },
        formatCertDays(row, col, cell) {
            var t1 = moment(row["StartAt"], "YYYY-MM-DD HH:mm:ss");
            var t2 = moment(row["EndAt"], "YYYY-MM-DD HH:mm:ss");
            if(t1 && t2 && t1.year() > 1 && t2.year() > 1) {
                return `${t1.format("YYYY-MM-DD")} ~ ${t2.format("YYYY-MM-DD")}`;
            }
            return "-";
        },
        formatCommonName(row, col, cell) {
            var ret = cell || "-";
            var list = ret.split("_");
            if(list.length == 2) {
                ret = list[0] || "-";
            }
            return ret;
        },
        getSMSList() {
            if (this.smsserial == "") {
                $.get("/api/v1/sms/list").then(ret => {
                    this.smss = ret;
                    if (ret.length > 0) {
                        this.sms = ret[0];
                        this.smsserial = ret[0].Serial;
                    }
                    this.getSMSInfo();
                    if (this.smsserial == "") {
                        this.timer = setTimeout(() => {
                            this.getSMSList();
                        }, 1000);
                    } else {
                        this.smstip = "流媒体服务尚未启动";
                    }
                })
            }
        },
        getSMSInfo() {
            if (this.smsserial != "") {
                $.get("/api/v1/sms/getbaseconfig", {
                    serial: this.smsserial,
                }).then(ret => {
                    this.smsbaseconfig = ret;
                    this.smsbaseconfig["RTSPPort"] = this.smsbaseconfig["RTSPPort"] || "";
                    this.smsbaseconfig["RTMPPort"] = this.smsbaseconfig["RTMPPort"] || "";
                    this.smsbaseconfig["CleanOverDays"] = this.smsbaseconfig["CleanOverDays"] || "";
                    this.smsbaseconfig["PreSerial"] = this.smsserial;
                    this.remoteSMSData = JSON.stringify(this.smsbaseconfig);
                })
            }
        },
        smschange() {
            this.getSMSInfo();
        },
        onMediaTransportChange(label) {
            if("UDP" == label) {
                this.MediaTransportMode = "passive";
            }
        },
    }, //-- methods
    computed: {
        ...mapState([ "logoText", "logoMiniText", "menus", "serverInfo" ]),
        isBasicNoChange() {
            var localeData = JSON.stringify(this.getBasicCommitObject());
            return this.remoteBasicData.localeCompare(localeData) == 0;
        },
        isSMSNoChange() {
            this.smsbaseconfig["PreSerial"] = this.smsserial;
            var localeData = JSON.stringify(this.smsbaseconfig);
            return this.remoteSMSData.localeCompare(localeData) == 0;
        }
    }
};
</script>

<style lang="less" scoped>
@import url(~assets/styles/variables.less);

.fa.cfg-cert-export,
.fa.cfg-black-list,
.fa.cfg-white-list {
    font-size: 14px;
    line-height: 24px;
    display: inline-block;
    color: @base;
}
</style>

<style lang="less">
#map-uploader {
    .el-upload {
        width: 100%;
        .el-upload-dragger {
            width: 100%;
            height: auto;
            .el-icon-upload {
                margin: 10px;
            }
            .el-upload__text {
                margin-bottom: 10px;
            }
        }
    }
}
</style>
