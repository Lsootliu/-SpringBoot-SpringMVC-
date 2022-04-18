# -SpringBoot-SpringMVC-
基于SpringBoot和SpringMVC、xml、jsp、mybatis、html的信息管理系统
控制层：

/**
 * Copyright &copy; 2012-2016 <a href="https://github.com/thinkgem/jeesite">JeeSite</a> All rights reserved.
 */
package com.thinkgem.jeesite.modules.liu.web;

import java.text.SimpleDateFormat;
import java.util.ArrayList;
import java.util.HashMap;
import java.util.List;

import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import javax.validation.ConstraintViolationException;

import org.apache.catalina.util.URLEncoder;
import org.apache.commons.collections.map.HashedMap;
import org.apache.shiro.authz.annotation.RequiresPermissions;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Controller;
import org.springframework.ui.Model;
import org.springframework.web.bind.annotation.ModelAttribute;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestMethod;
import org.springframework.web.bind.annotation.RequestParam;
import org.springframework.web.multipart.MultipartFile;
import org.springframework.web.servlet.mvc.support.RedirectAttributes;

import com.fasterxml.jackson.databind.module.SimpleAbstractTypeResolver;
import com.github.abel533.echarts.series.Map;
import com.google.common.collect.Lists;
import com.thinkgem.jeesite.common.beanvalidator.BeanValidators;
import com.thinkgem.jeesite.common.config.Global;
import com.thinkgem.jeesite.common.persistence.Page;
import com.thinkgem.jeesite.common.web.BaseController;
import com.thinkgem.jeesite.common.utils.DateUtils;
import com.thinkgem.jeesite.common.utils.StringUtils;
import com.thinkgem.jeesite.common.utils.excel.ExportExcel;
import com.thinkgem.jeesite.common.utils.excel.ImportExcel;
import com.thinkgem.jeesite.modules.gh.entity.Progress;
import com.thinkgem.jeesite.modules.gh.entity.ProjectInfo;
import com.thinkgem.jeesite.modules.liu.entity.TestOne;
import com.thinkgem.jeesite.modules.liu.entity.Workbench;
import com.thinkgem.jeesite.modules.liu.service.WorkbenchService;
import com.thinkgem.jeesite.modules.stat.entity.SettlementDetails;
import com.thinkgem.jeesite.modules.stat.entity.vo.SettlementDetailsImExport;
import com.thinkgem.jeesite.modules.sys.entity.User;
import com.thinkgem.jeesite.modules.sys.service.SystemService;
import com.thinkgem.jeesite.modules.sys.utils.UserUtils;

/**
 * 设备信息管理Controller
 * @author 
 * @version 
 */
@Controller
@RequestMapping(value = "${adminPath}/liu/Workbench")
public class WorkbenchController extends BaseController {


	@Autowired
	private WorkbenchService workbenchService;
	
	@ModelAttribute
	public Workbench get(@RequestParam(required=false) String id) {
		Workbench entity = null;
		if (StringUtils.isNotBlank(id)){
			entity = workbenchService.get(id);
		}
		if (entity == null){
			entity = new Workbench();
		}
		return entity;
	}
	
	@RequiresPermissions("liu:Workbench:view")
	@RequestMapping(value = {"list", ""})
	public String list(Workbench workbench, HttpServletRequest request, HttpServletResponse response, Model model) {
		Page<Workbench> page = workbenchService.findPage(new Page<Workbench>(request, response), workbench); 
		model.addAttribute("page", page);
		model.addAttribute("Workbench", workbench);
		return "modules/liu/workbenchList";
	}

	@RequiresPermissions("liu:Workbench:view")
	@RequestMapping(value = "form")
	public String form(Workbench workbench, Model model) {
		model.addAttribute("Workbench", workbench);
		return "modules/liu/workbenchForm";
	}
	
	/**
	 * 设备履历表
	 * 
	 * @param Workbench
	 * @param model
	 * @return
	 */
	@RequestMapping(value = "doc")
	public String WorkbenchDoc(Workbench workbench, Model model) {
		model.addAttribute("Workbench", workbench);
		return "modules/liu/workbenchDoc";
	}
	
	/**
	 * 设备基本信息概览
	 * 
	 * @param Workbench
	 * @param model
	 * @return
	 */
	@RequestMapping(value = "baseInfo")
	public String baseInfo(Workbench workbench, Model model) {
		model.addAttribute("Workbench", workbench);
		return "modules/liu/workbenchBaseInfo";
	}

	@RequiresPermissions("liu:Workbench:edit")
	@RequestMapping(value = "save")
	public String save(Workbench workbench, Model model, RedirectAttributes redirectAttributes) {
		if (!beanValidator(model, workbench)){
			return form(workbench, model);
		}
		workbenchService.save(workbench);
		addMessage(redirectAttributes, "保存设备信息管理成功");
		return "redirect:"+Global.getAdminPath()+"/liu/Workbench/?repage";
	}
	
	@RequiresPermissions("liu:Workbench:edit")
	@RequestMapping(value = "delete")
	public String delete(Workbench workbench, RedirectAttributes redirectAttributes) {
		workbenchService.delete(workbench);
		addMessage(redirectAttributes, "删除数据成功");
		return "redirect:"+Global.getAdminPath()+"/liu/Workbench/?repage";
	
	}

	/**
	 * 导出用户数据
	 * @param user
	 * @param request
	 * @param response
	 * @param redirectAttributes
	 * @return
	 */


	@RequestMapping(value = "btnexport", method=RequestMethod.POST)
    public String export(Workbench workbench, HttpServletRequest request, HttpServletResponse response, RedirectAttributes redirectAttributes) {
		try {
            String export = "用户数据"+DateUtils.getDate("yyyyMMddHHmmss")+".xlsx";
        	/*Page<Workbench> page = workbenchService.findPage(new Page<Workbench>(request, response), workbench); */
        	 List<Workbench> list = workbenchService.findList(workbench);
     		new ExportExcel("用户数据", Workbench.class).setDataList(list).write(response, export).dispose();
        	/*model.addAttribute("page", page);
    		model.addAttribute("Workbench", workbench);*/
    		return null;
    	
		} catch (Exception e) {
			addMessage(redirectAttributes, "导出用户失败！失败信息："+e.getMessage());
		}
		return "redirect:" + adminPath + "/liu/Workbench/list?repage";
    }
	
	
	/**
	 * 导入用户数据
	 * @param file
	 * @param redirectAttributes
	 * @return
	 */
 @RequestMapping(value = "btnImport", method=RequestMethod.POST)
    public String export(MultipartFile file, RedirectAttributes redirectAttributes) {
		if(Global.isDemoMode()){
			addMessage(redirectAttributes, "演示模式，不允许操作！");
			return "redirect:" + adminPath + "/liu/workbench/list?repage";
		}
		try {
			int successNum = 0;
			int failureNum = 0;
			StringBuilder failureMsg = new StringBuilder();
			ImportExcel ei = new ImportExcel(file, 1, 0);
			List<Workbench> list = ei.getDataList(Workbench.class);
			for (Workbench workbench : list){
				
				try{
						workbenchService.save(workbench);
						successNum++;
				}catch(ConstraintViolationException ex){
					failureMsg.append("<br/>登录名 "+workbench.getName()+" 导入失败：");
					List<String> workbenchList = BeanValidators.extractPropertyAndMessageAsList(ex, ": ");
					for (String message : workbenchList){
						failureMsg.append(message+"; ");
						failureNum++;
					}
				}catch (Exception ex) {
					failureMsg.append("<br/>登录名 "+workbench.getName()+" 导入失败："+ex.getMessage());
				}
			}
			if (failureNum>0){
				failureMsg.insert(0, "，失败 "+failureNum+" 条用户，导入信息如下：");
			}
			addMessage(redirectAttributes, "已成功导入 "+successNum+" 条用户"+failureMsg);
		} catch (Exception e) {
			addMessage(redirectAttributes, "导入用户失败！失败信息："+e.getMessage());
		}
		return "redirect:" + adminPath + "/liu/Workbench/list?repage";
    }
 
 
	

	
}














服务层：

/**
 * Copyright &copy; 2012-2016 <a href="https://github.com/thinkgem/jeesite">JeeSite</a> All rights reserved.
 */
package com.thinkgem.jeesite.modules.liu.service;

import java.util.List;

import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.Transactional;

import com.thinkgem.jeesite.common.persistence.Page;
import com.thinkgem.jeesite.common.service.CrudService;
import com.thinkgem.jeesite.modules.liu.entity.Workbench;
import com.thinkgem.jeesite.modules.liu.dao.WorkbenchDao;
import com.thinkgem.jeesite.modules.stat.entity.SettlementDetails;

/**
 * 设备信息管理Service
 * @author 
 * @version 
 */
@Service
@Transactional(readOnly = true)
public class WorkbenchService extends CrudService<WorkbenchDao, Workbench> {

	
	public Workbench get(String id) {
		return super.get(id);
	}
	
	public List<Workbench> findList(Workbench workbench) {
		return super.findList(workbench);
	}
	
	public Page<Workbench> findPage(Page<Workbench> page, Workbench workbench) {
		return super.findPage(page, workbench);
	}
	
	@Transactional(readOnly = false)
	public void save(Workbench workbench) {
		super.save(workbench);
	}
	
	@Transactional(readOnly = false)
	public void delete(Workbench workbench) {
		super.delete(workbench);
	}



}







实体层：

/**
 * Copyright &copy; 2012-2016 <a href="https://github.com/thinkgem/jeesite">JeeSite</a> All rights reserved.
 */
package com.thinkgem.jeesite.modules.liu.entity;

import org.hibernate.validator.constraints.Length;
import java.util.Date;
import com.fasterxml.jackson.annotation.JsonFormat;

import com.thinkgem.jeesite.common.persistence.DataEntity;
import com.thinkgem.jeesite.common.utils.excel.annotation.ExcelField;

/**
 * 设备信息管理Entity
 * @author lhq
 * @version 2017-05-07 2022.03.15
 */
public class Workbench extends DataEntity<Workbench> {
	
	private static final long serialVersionUID = 1L;
//	private String id;
	private String name;		// 设备名称
	private String type;		// 规格型号/代号
	private String code;		// 编号
	private String batch;		// 批次
	private String unite;		// 单位
	private String number;		// 数量
	private Date outDate;		// 出厂时间
	private Date pretendDate;		// 列装时间
	private String factory;		// 生产厂家
	private String manageDepart;		// 管理单位
	private String location;		// 存放地点
	private String quality;		// 质量状况
	private String comments;		// 备注
	
	
	
	public Workbench() {
		super();
	}

	public Workbench (String id){
		super(id);
	}
	@ExcelField(title="设备名称", align=2, sort=1)
	@Length(min=0, max=64, message="设备名称长度必须介于 0 和 64 之间")
	public String getName() {
		return name;
	}

	public void setName(String name) {
		this.name = name;
	}
	@ExcelField(title="规格型号/代号", align=2, sort=2)
	@Length(min=0, max=64, message="规格型号/代号长度必须介于 0 和 64 之间")
	public String getType() {
		return type;
	}

	public void setType(String type) {
		this.type = type;
	}
	@ExcelField(title="编号", align=2, sort=3)
	@Length(min=0, max=64, message="编号长度必须介于 0 和 64 之间")
	public String getCode() {
		return code;
	}

	public void setCode(String code) {
		this.code = code;
	}
	@ExcelField(title="批次", align=2, sort=4)
	@Length(min=0, max=256, message="批次长度必须介于 0 和 256 之间")
	public String getBatch() {
		return batch;
	}

	public void setBatch(String batch) {
		this.batch = batch;
	}
	@ExcelField(title="单位", align=2, sort=5)
	@Length(min=0, max=64, message="单位长度必须介于 0 和 64 之间")
	public String getUnite() {
		return unite;
	}

	public void setUnite(String unite) {
		this.unite = unite;
	}
	@ExcelField(title="数量", align=2, sort=6)
	@Length(min=0, max=11, message="数量长度必须介于 0 和 11 之间")
	public String getNumber() {
		return number;
	}

	public void setNumber(String number) {
		this.number = number;
	}
	@ExcelField(title="装列时间", align=2, sort=7)
	@JsonFormat(pattern = "yyyy-MM-dd HH:mm:ss")
	public Date getOutDate() {
		return outDate;
	}

	public void setOutDate(Date outDate) {
		this.outDate = outDate;
	}
	@ExcelField(title="出厂时间", align=2, sort=8)
	@JsonFormat(pattern = "yyyy-MM-dd HH:mm:ss")
	public Date getPretendDate() {
		return pretendDate;
	}

	public void setPretendDate(Date pretendDate) {
		this.pretendDate = pretendDate;
	}
	@ExcelField(title="出厂厂家", align=2, sort=9)
	@Length(min=0, max=64, message="生产厂家长度必须介于 0 和 64 之间")
	public String getFactory() {
		return factory;
	}

	public void setFactory(String factory) {
		this.factory = factory;
	}
	@ExcelField(title="单位", align=2, sort=10)
	@Length(min=0, max=64, message="管理单位长度必须介于 0 和 64 之间")
	public String getManageDepart() {
		return manageDepart;
	}

	public void setManageDepart(String manageDepart) {
		this.manageDepart = manageDepart;
	}
	@ExcelField(title="存放地点", align=2, sort=11)
	@Length(min=0, max=64, message="存放地点长度必须介于 0 和 64 之间")
	public String getLocation() {
		return location;
	}

	public void setLocation(String location) {
		this.location = location;
	}
	@ExcelField(title="质量状况", align=2, sort=12)
	@Length(min=0, max=256, message="质量状况长度必须介于 0 和 256 之间")
	public String getQuality() {
		return quality;
	}

	public void setQuality(String quality) {
		this.quality = quality;
	}
	@ExcelField(title="备注", align=2, sort=13)
	@Length(min=0, max=512, message="备注长度必须介于 0 和 512 之间")
	public String getComments() {
		return comments;
	}

	public void setComments(String comments) {
		this.comments = comments;
	}

//	public String getworkbench() {
//		
//		return id ;
//	}
//	public void setworkbench(String workbench) {
//		this.id=workbench;
//	}
	
	
}



接口层：


/**
 * Copyright &copy; 2012-2016 <a href="https://github.com/thinkgem/jeesite">JeeSite</a> All rights reserved.
 */
package com.thinkgem.jeesite.modules.liu.dao;

import com.thinkgem.jeesite.common.persistence.CrudDao;
import com.thinkgem.jeesite.common.persistence.annotation.MyBatisDao;
import com.thinkgem.jeesite.modules.liu.entity.Workbench;

/**
 * 设备信息管理DAO接口
 * @author 
 * @version 
 */
@MyBatisDao
public interface WorkbenchDao extends CrudDao<Workbench> {
	
}









数据库：



<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.thinkgem.jeesite.modules.liu.dao.WorkbenchDao">
    
	<sql id="workbenchColumns">
		a.gzt_id AS "id",
		a.gzt_name AS "name",
		a.gzt_type AS "type",
		a.gzt_code AS "code",
		a.gzt_batch AS "batch",
		a.gzt_unite AS "unite",
		a.gzt_number AS "number",
		a.gzt_outDate AS "outDate",
		a.gzt_pretendDate AS "pretendDate",
		a.gzt_factory AS "factory",
		a.gzt_manageDepart AS "manageDepart",
		a.gzt_location AS "location",
		a.gzt_quality AS "quality",
		a.gzt_comments AS "comments" 
	</sql>
	
	<sql id="workbenchJoins">
		
	</sql>
    <!-- 单个查询用户数目 -->
	<select id="get" resultType="Workbench">
		SELECT 
			<include refid="workbenchColumns"/>
		FROM liu_gzt a
		<include refid="workbenchJoins"/>
		WHERE a.gzt_id = #{id}
	</select>
	
	
	<select id="findList" resultType="Workbench">
		SELECT 
			<include refid="workbenchColumns"/>
		FROM liu_gzt a
		<include refid="workbenchJoins"/>
		<where><!--进行查询判断-->
			
			<if test="name != null and name != ''">
					AND a.gzt_name = #{name}
			</if>
			
			<if test="unite != null and unite != ''">
				AND a.gzt_unite = #{unite}
			</if> 
			 <if test="unite != null and unite != ''">
				AND a.gzt_unite LIKE 
					<if test="dbName == 'mysql'">concat('%',#{unite},'%')</if>
			</if> 	
			
			
		</where>
	</select>

	<select id="findAllList" resultType="Workbench">
		SELECT 
			<include refid="workbenchColumns"/>
		FROM liu_gzt a
		<include refid="workbenchJoins"/>
		<where>
			
		</where>		
	</select>
	<!-- 插入用户数据 -->
	<insert id="insert">
		INSERT INTO liu_gzt(
		
			gzt_name,
			gzt_type,
			gzt_code,
			gzt_batch,
			gzt_unite,
			gzt_number,
			gzt_outDate,
			gzt_pretendDate,
			gzt_factory,
			gzt_manageDepart,
 			gzt_location,
			gzt_quality,
			gzt_comments 
		) VALUES (
			
			#{name},
			#{type},
			#{code},
			#{batch},
			#{unite},
			#{number},
			#{outDate},
			#{pretendDate},
			#{factory},
			#{manageDepart},
			#{location},
			#{quality},
			#{comments} 
		)
	</insert>
	
	<!--更新登录信息，如登录IP、登录时间  -->
	<update id="update">
		UPDATE liu_gzt SET 	
	
			gzt_name = #{name},
			gzt_type = #{type},
			gzt_code = #{code},
			gzt_batch = #{batch},
			gzt_unite = #{unite},
			gzt_number = #{number},
			gzt_outDate = #{outDate},
			gzt_pretendDate = #{pretendDate},
			gzt_factory = #{factory},
			gzt_manageDepart = #{manageDepart},
			gzt_location = #{location},
			gzt_quality = #{quality},
			gzt_comments = #{comments}
		WHERE gzt_id = #{id}
	</update>
	
	<update id="delete">
		DELETE FROM liu_gzt
		WHERE gzt_id = #{id}
	</update>
	
</mapper>








List页面：



<%@ page contentType="text/html;charset=UTF-8" %>
<%@ include file="/WEB-INF/views/include/taglib.jsp"%>
<html>
<head>



	<title>设备信息管理管理</title>
	<meta name="decorator" content="default"/>
	<script type="text/javascript">
	
	$(document).ready(function() {
		$("#btnExport").click(function(){
			top.$.jBox.confirm("确认要导出用户数据吗？","系统提示",function(v,h,f){
				if(v=="ok"){
					$("#searchForm").attr("action","${ctx}/liu/Workbench/btnexport");
					$("#searchForm").submit();
				}
			},{buttonsFocus:1});
			top.$('.jbox-body .jbox-icon').css('top','55px');
		});
		$("#btnImport").click(function(){
			$.jBox($("#importBox").html(), {title:"导入数据", buttons:{"关闭":true}, 
				bottomText:"导入文件不能超过5M，仅允许导入“xls”或“xlsx”格式文件！"});
		});
	});
	
	/* 
	$(document).ready(function() {
		$("#btnImport").click(function(){
			
			
			
			$("#btnExport").click(function(){
				  top.$.jBox.confirm("确认要导出用户数据吗？","系统提示",function(v,h,f){
					  if(v=="ok"){
							$("#searchForm").attr("action","${ctx}/liu/Workbench/btnexport");
							$("#searchForm").submit();
						}
					},{buttonsFocus:1});
					top.$('.jbox-body .jbox-icon').css('top','55px');
					}); 
		});
			
			$.jBox($("#importBox").html(), {title:"导入数据", buttons:{"关闭":true}, 
				bottomText:"导入文件不能超过5M，仅允许导入“xls”或“xlsx”格式文件！"});
			
		}); */
	
		
		
	function page(n,s){
		if(n)$("#pageNo").val(n);
		if (s)$("#pageSize").val(s);
		$("#searchForm").submit();
	 	$("#searchForm").attr("action","${ctx}/liu/Workbench/list"); 
    	return false;
    }
	</script> 
</head>
<body>
	<%-- <ul class="nav nav-tabs">
		<li class="active"><a href="${ctx}/liu/workbench/">设备信息管理列表</a></li>
		<shiro:hasPermission name="liu:workbench:edit"><li><a href="${ctx}/liu/workbench/form">设备信息管理添加</a></li></shiro:hasPermission>
	</ul> --%>
	
	<div style="padding-bottom:10px;padding-top:5px;width:100%;">
		<table width="100%">
		<tr>
		<td>
		<div class="function_title">
			<strong>设备信息管理列表</strong>
		</div>
		</td>
		<td>
		<div style="float:left;width:100%;text-align:right;">
			<a href="${ctx}/liu/Workbench/form" class="btn btn-primary btn-lg" role="button">增加设备信息</a>
		</div>
		</td>
		</tr>
		</table>
	</div>
	
	<div id="importBox" class="hide">
		<form id="importForm" action="${ctx}/liu/Workbench/btnImport" method="post" enctype="multipart/form-data"
			class="form-search" style="padding-left:20px;text-align:center;" onsubmit="loading('正在导入，请稍等...');"><br/>
			<input id="uploadFile" name="file" type="file" style="width:330px"/><br/><br/>　　
			<input id="btnImportSubmit" class="btn btn-primary" type="submit"  	value="   导    入   "/>
			<a href="${ctx}/liu/workbench/import/template">下载模板</a>
		</form>
		<%-- <form id="btnExport" action="${ctx}/liu/workbench/export" method="post" enctype="multipart/form-data"
			class="form-search" style="padding-left:20px;text-align:center;" onsubmit="loading('正在导出，请稍等...');"><br/>
			<input id="btnExport" class="btn btn-primary" type="button" value="   导    出   "/>
		</form> --%>
		
	</div>
	<form:form id="searchForm" modelAttribute="workbench" action="${ctx}/liu/Workbench/" method="post" class="breadcrumb form-search">
		<input id="pageNo" name="pageNo" type="hidden" value="${page.pageNo}"/>
		<input id="pageSize" name="pageSize" type="hidden" value="${page.pageSize}"/>
		<ul class="ul-form"  height="60px">
			<li>
				<label>设备类型：</label>
				<sys:treeselect id="parent" name="parent.id" value="${txCarBrand.parent.id}" labelName="parent.name" labelValue="${txCarBrand.parent.brandName}"
					title="父级编号" url="/tx/txCarBrand/treeData" extId="${txCarBrand.id}" cssClass="" allowClear="true"/>
			</li>
		<%-- 	<li><label>项目名称：</label>
				<form:select path="id" class="input-medium">
					<form:option value="" label="请选择项目名..."/>
					<form:options items="${bzfProjectInfoList}" itemLabel="name" itemValue="id" htmlEscape="false"/>
				</form:select>
			</li>
			 --%>
			
	<%-- 		<li><label>单位：</label>
				<form:select path="unite" class="input-medium">
					<form:option value="" label=""/>
					<form:options items="${fns:getDictList('liu_workbench_unite')}" itemLabel="label" itemValue="value" htmlEscape="false"/>
				</form:select><!--字典的使用  -->
			</li> --%>
			<li><label>单位：</label>
				<form:select path="unite" class="input-medium">
					<form:option value="" label=""/>
					<form:options items="${fns:getDictList('liu_workbench_unite')}" itemLabel="label" itemValue="value" htmlEscape="false"/>
				</form:select>
			</li>
		
			<li><label>设备名称：</label>
				<form:input path="name" htmlEscape="false" maxlength="75"  class="input-medium"/>
			</li>
			
			<li class="btns"><input id="btnSubmit" class="btn btn-primary" type="submit" value="查询"/></li>
			<li class="btns"><input id="btnImport" class="btn btn-primary" type="button" value="导入/导出"/></li>
		
			<li class="btns"><input id="btnExport" class="btn btn-primary" type="button" value="导出"/></li> 
			<li class="clearfix"></li>
		
			<li><label>装列时间时间</label>
			<input id="beginDate"  name="beginDate"  type="text" readonly="readonly" maxlength="75" class="input-medium Wdate" style="width:163px;"
				value="<fmt:formatDate value="${act.beginDate}" pattern="yyyy-MM-dd"/>"
					onclick="WdatePicker({dateFmt:'yyyy-MM-dd'});"/>
			<label>出厂时间</label>
			<input id="endDate" name="endDate" type="text" readonly="readonly" maxlength="75" class="input-medium Wdate" style="width:163px;"
				value="<fmt:formatDate value="${act.endDate}" pattern="yyyy-MM-dd"/>"
					onclick="WdatePicker({dateFmt:'yyyy-MM-dd'});"/> </li>
		</ul>
	</form:form>
	 <sys:message content="${message}"/>
	<table id="contentTable" class="table table-striped table-bordered table-condensed">
	
		<thead>
			<tr>
			
				<th>设备名称</th>
				<th>规格型号/代号</th>
				<th>编号</th>
				<th>批次</th>
				<th>单位</th>
				<th>出厂厂家</th>
				<th>数量</th>
				<th>列装时间</th>
				<th>出厂时间</th>
				<th>存放地点</th>
				<th>质量状况</th>
				
				<th>备注</th>
				<th>操作</th>
			</tr>
		</thead>
		<tbody>
		<c:forEach items="${page.list}" var="workbench"><!-- 页面值的展示 -->
		
			<tr>
				<td><a href="${ctx}/liu/workbench/form?id=${workbench.id}">
					${workbench.name}
				</a></td>
				<td>
					${workbench.type}
				</td>
				<td>
					${workbench.code}
				</td>
				<td>
					${workbench.batch}
				</td>
				<td>
					${workbench.unite}
				</td>
				<td>
					${workbench.factory}
				</td>
				<td>
					${workbench.number}
				</td>
				<td>
					<fmt:formatDate value="${workbench.outDate}" pattern="yyyy-MM-dd HH:mm:ss"/>
				</td>
				<td>
					<fmt:formatDate value="${workbench.pretendDate}" pattern="yyyy-MM-dd HH:mm:ss"/>
				</td>
				
				<td>
					${workbench.manageDepart}
				</td>
				<td>
					${workbench.quality}
				</td>
				<td>
					${workbench.comments}
				</td>
		
				<shiro:hasPermission name="liu:Workbench:edit"><td>
    				<a href="${ctx}/liu/Workbench/form?id=${workbench.id}">修改</a>
					<a href="${ctx}/liu/Workbench/delete?id=${workbench.id}&test.id=${testid}" onclick="return confirmx('确认要删除该违规情况登记吗？', this.href)">删除</a>
				</td></shiro:hasPermission>
			</tr>
		</c:forEach>
		</tbody>
		
	</table>
<div class="pagination">${page}</div> 
</body>
</html>







Form页面：


<%@ page contentType="text/html;charset=UTF-8" %>
<%@ include file="/WEB-INF/views/include/taglib.jsp"%>
<html>
<head>
	<title>设备信息管理管理</title>
	<meta name="decorator" content="default"/>
	<script type="text/javascript">
		$(document).ready(function() {
			
			//$("#name").focus();
			$("#inputForm").validate({
				submitHandler: function(form){
					loading('正在提交，请稍等...');
					form.submit();
				},
				errorContainer: "#messageBox",
				errorPlacement: function(error, element) {
					$("#messageBox").text("输入有误，请先更正。");
					if (element.is(":checkbox")||element.is(":radio")||element.parent().is(".input-append")){
						error.appendTo(element.parent().parent());
					} else {
						error.insertAfter(element);
					}
				}
			});
		});
	</script>
</head>
<body>
	<%-- <ul class="nav nav-tabs">
		<li><a href="${ctx}/ep/equiment/">设备信息管理列表</a></li>
		<li class="active"><a href="${ctx}/ep/equiment/form?id=${equiment.id}">设备信息管理<shiro:hasPermission name="ep:equiment:edit">${not empty equiment.id?'修改':'添加'}</shiro:hasPermission><shiro:lacksPermission name="ep:equiment:edit">查看</shiro:lacksPermission></a></li>
	</ul><br/> --%>
	<div style="padding-bottom:10px;padding-top:5px;width:100%;">
		<div class="function_title">
			<strong>${not empty carinfo.id?'修改':'添加'}设备信息</strong>
		</div>
	</div>
	
	<form:form id="inputForm" modelAttribute="Workbench" action="${ctx}/liu/Workbench/save" method="post" class="form-horizontal">
		<form:hidden path="id"/>
		<sys:message content="${message}"/>
		
		<table width="100%">
		<tr style="height:85px;">
			<td align="right">设备名称：  </td>
				<td><form:input path="name" htmlEscape="false" maxlength="74" class="input-xlarge "/></td>
			<td align="right">设备类型：</td>
			<td>
				<sys:treeselect id="name" name="parent.id" value="${txCarBrand.parent.id}" labelName="parent.gzt_name" labelValue="${txCarBrand.parent.brandName}"
				title="父级编号" url="/tx/txCarBrand/treeData" extId="${txCarBrand.id}" cssClass="" allowClear="true"/>
			</td>
			<td align="right" >规格型号/代号：</td>
			<td><form:input path="type" htmlEscape="false" maxlength="74" class="input-xlarge "/></td>
		</tr>
		<tr style="height:65px;">
			<td align="right">编号：</td>
				<td><form:input path="code" htmlEscape="false" maxlength="72" class="input-xlarge "/></td>
			<td align="right">批次：</td>
				<td><form:input path="batch" htmlEscape="false" maxlength="72" class="input-xlarge "/></td>
			<td align="right">单位：</td>
				<td><form:input path="unite" htmlEscape="false" maxlength="72" class="input-xlarge "/></td>
		</tr>
		<tr style="height:65px;">
			<td align="right">数量：</td>
				<td><form:input path="number" htmlEscape="false" maxlength="72" class="input-xlarge "/></td>
			<td align="right">出厂时间：</td>
				<td><input name="outDate" type="text" readonly="readonly" maxlength="20" class="input-medium Wdate "
					value="<fmt:formatDate value="${workbench.outDate}" pattern="yyyy-MM-dd"/>"
					onclick="WdatePicker({dateFmt:'yyyy-MM-dd',isShowClear:false});"/></td>
			<td align="right">列装时间：</td>
			<td><input name="pretendDate" type="text" readonly="readonly" maxlength="20" class="input-medium Wdate "
				value="<fmt:formatDate value="${workbench.pretendDate}" pattern="yyyy-MM-dd HH:mm:ss"/>"
				onclick="WdatePicker({dateFmt:'yyyy-MM-dd HH:mm:ss',isShowClear:false});"/></td>
		</tr>
		<tr style="height:65px;">
			<td align="right">生产厂家：</td>
				<td><form:input path="factory" htmlEscape="false" maxlength="74" class="input-xlarge "/></td>
				
			<td align="right">管理单位：</td>
				<td><form:input path="manageDepart" htmlEscape="false" maxlength="74" class="input-xlarge "/></td>
			<td align="right">存放地点：</td>
				<td><form:input path="location" htmlEscape="false" maxlength="74" class="input-xlarge "/></td>
			
		</tr>
		<tr style="height:35px;">
				<td align="right">质量状况：</td>
				<td colspan="2"><form:textarea path="quality" htmlEscape="false" rows="4" maxlength="256" class="input-xxlarge "/></td>
				<td align="right">备注：</td>
				<td colspan="2"><form:textarea path="comments" htmlEscape="false" rows="4" maxlength="512" class="input-xxlarge "/></td>
		</tr>
	</table>
			<div class="form-actions">
				<input id="btnSubmit" class="btn btn-primary" type="submit" value="保 存"/>&nbsp;
				<input id="btnCancel" class="btn" type="button" value="返 回" onclick="history.go(-1)"/>
			</div>
	</form:form>
</body>
</html>












