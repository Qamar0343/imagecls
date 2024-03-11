CREATE FUNCTION [dbo].[GetContentStatus]

(
  @IsPublished bit,
  @IsArchived bit,
  @IsDeleted bit,
  @PublishDateFrom datetime,
  @PublishDateTo datetime,
  @ArchiveDateFrom datetime,
  @ArchiveDateTo datetime
)
RETURNS nvarchar(100)
AS
BEGIN
declare @result_Deleted nvarchar(100)='';
declare @result_Archived nvarchar(100)='';
declare @result_Published nvarchar(100)='';

	select 
	
	@result_Deleted=
	 (
	 	 case 
		 when @IsDeleted=1 then ';Deleted;' else ';Active;'
		 end
	 ),
	 
	 @result_Archived =
	 (
		 case
		 when @IsArchived=1 then ';Archived;' 
	 	 when @IsArchived=0 and
			   (  
				  (@ArchiveDateFrom is not null and @ArchiveDateTo is not null  and @ArchiveDateFrom<=GETUTCDATE() and @ArchiveDateTo>=GETUTCDATE())
				 or
				  (@ArchiveDateFrom is not null and @ArchiveDateTo is  null and   @ArchiveDateFrom<=GETUTCDATE())
	  			 or
				  (@ArchiveDateFrom is null and @ArchiveDateTo is not null and   @ArchiveDateTo>=GETUTCDATE())

			  )  
			  then ';Archived;'
			  else ';NotArchived;'
	  	end  
     ),
	 @result_Published=
	 (
	    case
	    when @IsPublished=1 then ';Published;'
	 	when @IsPublished=0 and
	  (  
	  (@PublishDateFrom is not null and @PublishDateTo is not null  and @PublishDateFrom<=GETUTCDATE() and @PublishDateTo>=GETUTCDATE())
	  or
	  (@PublishDateFrom is not null and @PublishDateTo is  null and   @PublishDateFrom<=GETUTCDATE())
	  	  or
	  (@PublishDateFrom is null and @PublishDateTo is not null and   @PublishDateTo>=GETUTCDATE())

	  ) then ';Published;'
	  else ';UnPublished;'
	  	end
		)

	 ;


	return CONCAT(@result_Deleted,@result_Published,@result_Archived) ;		
END
GO
/****** Object:  UserDefinedFunction [dbo].[GetProductConstant]    Script Date: 3/11/2024 8:29:35 AM ******/
SET ANSI_NULLS ON
GO
SET QUOTED_IDENTIFIER ON
GO
-- =============================================
-- Author:		<Author,,Name>
-- Create date: <Create Date, ,>
-- Description:	<Description, ,>
-- =============================================
CREATE FUNCTION [dbo].[GetProductConstant]
(
	-- Add the parameters for the function here
	@langId int,
	@optionValue nchar(5)
)
RETURNS nvarchar(max)
AS
BEGIN
	-- Declare the return variable here
	DECLARE @result nvarchar(max)
	
	select @result = Name from dbo.Translations where LanguageId = @langId and RefId = @optionValue

	-- Return the result of the function
	RETURN @result

END
GO
/****** Object:  UserDefinedFunction [dbo].[GetTranslation]    Script Date: 3/11/2024 8:29:35 AM ******/
SET ANSI_NULLS ON
GO
SET QUOTED_IDENTIFIER ON
GO


CREATE FUNCTION [dbo].[GetTranslation]
(	
    @xml XML,
	@col NVARCHAR(MAX)	,
	@attr NVARCHAR(MAX)	
)
RETURNS NVARCHAR(MAX)
AS
BEGIN
 DECLARE @result NVARCHAR(MAX);
 SELECT @result = @xml.value('(//*[local-name() = sql:variable("@attr")])[1]','nvarchar(max)');
 
 if(@result is null or LEN(ltrim(rtrim(@result))) = 0)
   set @result = @col;

 RETURN @result;
END



GO
/****** Object:  Table [dbo].[Translations]    Script Date: 3/11/2024 8:29:35 AM ******/
SET ANSI_NULLS ON
GO
SET QUOTED_IDENTIFIER ON
GO
CREATE TABLE [dbo].[Translations](
	[Id] [int] IDENTITY(1,1) NOT NULL,
	[RefId] [varchar](50) NOT NULL,
	[RefType] [char](1) NOT NULL,
	[Name] [nvarchar](2000) NULL,
	[Fields] [xml] NULL,
	[LanguageId] [int] NOT NULL,
	[LanguageGroupId] [uniqueidentifier] NOT NULL,
	[DateCreated] [datetime] NOT NULL,
	[CreatedBy] [nvarchar](128) NOT NULL,
	[LastUpdated] [datetime] NOT NULL,
	[LastUpdatedBy] [nvarchar](128) NOT NULL,
 CONSTRAINT [PK_Translations] PRIMARY KEY CLUSTERED 
(
	[Id] ASC
)WITH (PAD_INDEX = OFF, STATISTICS_NORECOMPUTE = OFF, IGNORE_DUP_KEY = OFF, ALLOW_ROW_LOCKS = ON, ALLOW_PAGE_LOCKS = ON, OPTIMIZE_FOR_SEQUENTIAL_KEY = OFF) ON [PRIMARY]
) ON [PRIMARY] TEXTIMAGE_ON [PRIMARY]
GO
/****** Object:  Table [dbo].[Categories]    Script Date: 3/11/2024 8:29:35 AM ******/
SET ANSI_NULLS ON
GO
SET QUOTED_IDENTIFIER ON
GO
CREATE TABLE [dbo].[Categories](
	[Id] [int] IDENTITY(1,1) NOT NULL,
	[ParentId] [int] NULL,
	[Name] [nvarchar](256) NOT NULL,
	[Description] [nvarchar](max) NULL,
	[DisplayOrder] [int] NOT NULL,
	[IsPublished] [bit] NOT NULL,
	[IsDeleted] [bit] NOT NULL,
	[DateCreated] [datetime] NOT NULL,
	[CreatedBy] [nvarchar](128) NOT NULL,
	[LastUpdated] [datetime] NULL,
	[LastUpdatedBy] [nvarchar](128) NULL,
	[IntegrationId] [nvarchar](128) NULL,
	[Icon] [nvarchar](256) NULL,
	[IntegrationIdEN] [nvarchar](200) NULL,
	[SubIntegrationIds] [nvarchar](200) NULL,
	[SubIntegrationIdsEN] [nvarchar](200) NULL,
 CONSTRAINT [PK_Categories] PRIMARY KEY CLUSTERED 
(
	[Id] ASC
)WITH (PAD_INDEX = OFF, STATISTICS_NORECOMPUTE = OFF, IGNORE_DUP_KEY = OFF, ALLOW_ROW_LOCKS = ON, ALLOW_PAGE_LOCKS = ON, OPTIMIZE_FOR_SEQUENTIAL_KEY = OFF) ON [PRIMARY]
) ON [PRIMARY] TEXTIMAGE_ON [PRIMARY]
GO
/****** Object:  View [dbo].[TranslatedCategories]    Script Date: 3/11/2024 8:29:35 AM ******/
SET ANSI_NULLS ON
GO
SET QUOTED_IDENTIFIER ON
GO


CREATE VIEW [dbo].[TranslatedCategories]
AS
SELECT        dbo.Categories.Id, dbo.Categories.ParentId, dbo.Categories.DisplayOrder, dbo.GetTranslation(dbo.Translations.Fields, dbo.Categories.Name, N'Name') AS Name, dbo.GetTranslation(dbo.Translations.Fields, 
                         dbo.Categories.Description, N'Description') AS Description, dbo.Categories.IsPublished, dbo.Categories.IsDeleted, dbo.Categories.DateCreated, dbo.Categories.CreatedBy, dbo.Categories.LastUpdated, 
                         dbo.Categories.LastUpdatedBy, dbo.Translations.LanguageId, dbo.Categories.IntegrationId, dbo.Categories.Icon, dbo.Categories.IntegrationIdEN,dbo.Categories.SubIntegrationIds,dbo.Categories.SubIntegrationIdsEN
FROM            dbo.Categories LEFT OUTER JOIN
                         dbo.Translations ON CONVERT(VARCHAR(50), dbo.Categories.Id) = dbo.Translations.RefId AND dbo.Translations.RefType = '_'
GO
/****** Object:  Table [dbo].[ContentTypes]    Script Date: 3/11/2024 8:29:35 AM ******/
SET ANSI_NULLS ON
GO
SET QUOTED_IDENTIFIER ON
GO
CREATE TABLE [dbo].[ContentTypes](
	[Id] [int] IDENTITY(1,1) NOT NULL,
	[Name] [nvarchar](256) NOT NULL,
	[Description] [nvarchar](2048) NULL,
	[Type] [char](1) NOT NULL,
	[URLPattern] [nvarchar](1000) NULL,
	[DefaultImage] [xml] SPARSE  NULL,
	[DefaultClassName] [nvarchar](128) NULL,
	[Attributes] [xml] SPARSE  NULL,
	[CustomCode] [nvarchar](max) NULL,
	[Configs] [xml] SPARSE  NULL,
	[FeedbackAttributes] [xml] SPARSE  NULL,
	[DateCreated] [datetime] NOT NULL,
	[LastUpdateDate] [datetime] NOT NULL,
	[CreatedBy] [nvarchar](128) NULL,
	[LastUpdatedBy] [nvarchar](128) NULL,
	[IsProtected] [bit] NOT NULL,
	[IsPublished] [bit] NOT NULL,
	[IsDeleted] [bit] NOT NULL,
 CONSTRAINT [PK_ContentTypes] PRIMARY KEY CLUSTERED 
(
	[Id] ASC
)WITH (PAD_INDEX = OFF, STATISTICS_NORECOMPUTE = OFF, IGNORE_DUP_KEY = OFF, ALLOW_ROW_LOCKS = ON, ALLOW_PAGE_LOCKS = ON, OPTIMIZE_FOR_SEQUENTIAL_KEY = OFF) ON [PRIMARY]
) ON [PRIMARY] TEXTIMAGE_ON [PRIMARY]
GO
/****** Object:  View [dbo].[TranslatedContentTypes]    Script Date: 3/11/2024 8:29:35 AM ******/
SET ANSI_NULLS ON
GO
SET QUOTED_IDENTIFIER ON
GO


CREATE VIEW [dbo].[TranslatedContentTypes]
AS
SELECT        dbo.ContentTypes.Id, dbo.GetTranslation(dbo.Translations.Fields, dbo.ContentTypes.Name, N'Name') AS Name, dbo.GetTranslation(dbo.Translations.Fields, dbo.ContentTypes.Description, N'Description') AS Description, 
                         dbo.ContentTypes.Type, dbo.ContentTypes.URLPattern, dbo.ContentTypes.DefaultImage, dbo.ContentTypes.DefaultClassName, dbo.ContentTypes.Attributes, dbo.ContentTypes.CustomCode, dbo.ContentTypes.Configs, 
                         dbo.ContentTypes.FeedbackAttributes, dbo.ContentTypes.DateCreated, dbo.ContentTypes.LastUpdateDate, dbo.ContentTypes.CreatedBy, dbo.ContentTypes.LastUpdatedBy, dbo.ContentTypes.IsProtected, 
                         dbo.ContentTypes.IsPublished, dbo.ContentTypes.IsDeleted, dbo.Translations.LanguageId
FROM            dbo.ContentTypes LEFT OUTER JOIN
                         dbo.Translations ON CONVERT(VARCHAR(50), dbo.ContentTypes.Id) = dbo.Translations.RefId AND dbo.Translations.RefType = '.'
GO
/****** Object:  Table [dbo].[Forms]    Script Date: 3/11/2024 8:29:35 AM ******/
SET ANSI_NULLS ON
GO
SET QUOTED_IDENTIFIER ON
GO
CREATE TABLE [dbo].[Forms](
	[Id] [int] IDENTITY(1,1) NOT NULL,
	[Title] [nvarchar](1000) NOT NULL,
	[IntroText] [nvarchar](max) NULL,
	[FullText] [nvarchar](max) NULL,
	[Slug] [nvarchar](128) NULL,
	[IsArchived] [bit] NOT NULL,
	[Images] [xml] SPARSE  NULL,
	[FormAttributes] [xml] SPARSE  NULL,
	[Configs] [xml] SPARSE  NULL,
	[MetaPageTitle] [nvarchar](1024) NULL,
	[MetaDescription] [nvarchar](2048) NULL,
	[MetaKeywords] [nvarchar](2048) NULL,
	[WorkflowStatus] [nvarchar](128) SPARSE  NULL,
	[WorkflowSummary] [xml] SPARSE  NULL,
	[CategoryId] [int] SPARSE  NULL,
	[ViewsCount] [int] SPARSE  NULL,
	[LikesCount] [int] NOT NULL,
	[RateAverage] [float] NOT NULL,
	[DislikesCount] [int] NOT NULL,
	[PublishDateFrom] [datetime] NULL,
	[PublishDateTo] [datetime] NULL,
	[DateCreated] [datetime] NOT NULL,
	[LastUpdated] [datetime] NOT NULL,
	[LastUpdatedBy] [nvarchar](128) NOT NULL,
	[CreatedBy] [nvarchar](128) NOT NULL,
	[IsSticky] [bit] NOT NULL,
	[IsPublished] [bit] NOT NULL,
	[IsDeleted] [bit] NOT NULL,
	[GenerateBarcode] [bit] NOT NULL,
	[IsLibrary] [bit] NOT NULL,
 CONSTRAINT [PK_ic_Forms] PRIMARY KEY CLUSTERED 
(
	[Id] ASC
)WITH (PAD_INDEX = OFF, STATISTICS_NORECOMPUTE = OFF, IGNORE_DUP_KEY = OFF, ALLOW_ROW_LOCKS = ON, ALLOW_PAGE_LOCKS = ON, OPTIMIZE_FOR_SEQUENTIAL_KEY = OFF) ON [PRIMARY]
) ON [PRIMARY] TEXTIMAGE_ON [PRIMARY]
GO
/****** Object:  View [dbo].[TranslatedForms]    Script Date: 3/11/2024 8:29:35 AM ******/
SET ANSI_NULLS ON
GO
SET QUOTED_IDENTIFIER ON
GO


CREATE VIEW [dbo].[TranslatedForms]
AS
SELECT      
      dbo.Forms.Id
      ,dbo.GetTranslation(dbo.Translations.Fields, dbo.Forms.Title, N'Title') AS Title
      ,dbo.GetTranslation(dbo.Translations.Fields, dbo.Forms.IntroText, N'IntroText') AS IntroText
      ,dbo.GetTranslation(dbo.Translations.Fields, dbo.Forms.FullText, N'FullText') AS FullText
      ,dbo.Forms.Slug
      ,dbo.Forms.PublishDateFrom
      ,dbo.Forms.PublishDateTo
      ,dbo.Forms.DateCreated
      ,dbo.Forms.LastUpdated
      ,dbo.Forms.LastUpdatedBy
      ,dbo.Forms.CreatedBy
      ,dbo.Forms.IsArchived
      ,dbo.Forms.IsSticky
      ,dbo.Forms.IsPublished
      ,dbo.Forms.IsDeleted
      ,dbo.Forms.Images
      ,dbo.Forms.FormAttributes
      ,dbo.Forms.Configs
      ,dbo.Forms.CategoryId
      ,dbo.Forms.ViewsCount
      ,dbo.Forms.LikesCount
      ,dbo.Forms.DislikesCount
      ,dbo.Forms.RateAverage
      ,dbo.Forms.MetaPageTitle
      ,dbo.Forms.MetaDescription
      ,dbo.Forms.MetaKeywords
      ,dbo.Forms.WorkflowStatus
      ,dbo.Forms.WorkflowSummary
      ,dbo.Translations.LanguageId
      ,dbo.Forms.GenerateBarcode
	  ,dbo.Forms.IsLibrary

FROM          
dbo.Forms   LEFT OUTER JOIN dbo.Translations ON CONVERT(VARCHAR(50), dbo.Forms.Id) = dbo.Translations.RefId AND dbo.Translations.RefType = 'F'


GO
/****** Object:  Table [dbo].[Constants]    Script Date: 3/11/2024 8:29:35 AM ******/
SET ANSI_NULLS ON
GO
SET QUOTED_IDENTIFIER ON
GO
CREATE TABLE [dbo].[Constants](
	[Id] [int] IDENTITY(1,1) NOT NULL,
	[Name] [nvarchar](512) NOT NULL,
	[Value] [nvarchar](50) NOT NULL,
	[Ordering] [int] NOT NULL,
	[DateCreated] [datetime] NOT NULL,
	[CreatedBy] [nvarchar](128) NOT NULL,
	[LastUpdated] [datetime] NOT NULL,
	[LastUpdatedBy] [nvarchar](128) NOT NULL,
	[IsPublished] [bit] NOT NULL,
	[IsDeleted] [bit] NOT NULL,
	[GroupId] [int] NOT NULL,
	[ParentGroupId] [int] NULL,
	[ParentId] [int] NULL,
	[IntegrationId] [nvarchar](128) NULL,
	[Icon] [nvarchar](256) NULL,
 CONSTRAINT [PK_Constants] PRIMARY KEY CLUSTERED 
(
	[Id] ASC
)WITH (PAD_INDEX = OFF, STATISTICS_NORECOMPUTE = OFF, IGNORE_DUP_KEY = OFF, ALLOW_ROW_LOCKS = ON, ALLOW_PAGE_LOCKS = ON, OPTIMIZE_FOR_SEQUENTIAL_KEY = OFF) ON [PRIMARY]
) ON [PRIMARY]
GO
/****** Object:  View [dbo].[TranslatedConstants]    Script Date: 3/11/2024 8:29:35 AM ******/
SET ANSI_NULLS ON
GO
SET QUOTED_IDENTIFIER ON
GO

 

CREATE VIEW [dbo].[TranslatedConstants]
AS
SELECT        dbo.Constants.Id, ISNULL(dbo.Translations.Name, dbo.Constants.Name) AS Name, dbo.Constants.Value, dbo.Constants.Ordering, dbo.Translations.LanguageId, dbo.Constants.DateCreated, dbo.Constants.CreatedBy, 
                         dbo.Constants.LastUpdated, dbo.Constants.LastUpdatedBy, dbo.Constants.IsPublished, dbo.Constants.IsDeleted, dbo.Constants.GroupId, dbo.Constants.ParentId, dbo.Constants.IntegrationId, dbo.Constants.ParentGroupId, 
                         dbo.Constants.Icon
FROM            dbo.Constants LEFT OUTER JOIN
                         dbo.Translations ON dbo.Constants.Id = dbo.Translations.RefId AND dbo.Translations.RefType = 'C'
WHERE        (dbo.Constants.IsPublished = 1) AND (dbo.Constants.IsDeleted = 0)
GO
/****** Object:  Table [dbo].[Menus]    Script Date: 3/11/2024 8:29:35 AM ******/
SET ANSI_NULLS ON
GO
SET QUOTED_IDENTIFIER ON
GO
CREATE TABLE [dbo].[Menus](
	[Id] [int] IDENTITY(1,1) NOT NULL,
	[Name] [nvarchar](500) NOT NULL,
	[ShortName] [nvarchar](200) NULL,
	[Title] [nvarchar](500) NOT NULL,
	[Keywords] [nvarchar](1000) NULL,
	[Description] [nvarchar](1000) NULL,
	[MenuType] [char](1) NOT NULL,
	[Type] [char](1) NOT NULL,
	[MenuResources] [xml] NULL,
	[IntroText] [nvarchar](max) NULL,
	[PageText] [nvarchar](max) NULL,
	[Link] [nvarchar](1000) NULL,
	[LinkPattern] [nvarchar](1000) NULL,
	[Target] [char](1) NULL,
	[Ordering] [int] NOT NULL,
	[ClassName] [nvarchar](50) NULL,
	[Parent] [int] NULL,
	[SubLevel] [int] NULL,
	[DateCreated] [datetime] NOT NULL,
	[CreatedBy] [nvarchar](128) NOT NULL,
	[LastUpdated] [datetime] NOT NULL,
	[LastUpdatedBy] [nvarchar](128) NOT NULL,
	[IsHomepage] [bit] NOT NULL,
	[IsPublished] [bit] NOT NULL,
	[IsDeleted] [bit] NOT NULL,
	[AuthorizedFunc] [nvarchar](max) NULL,
	[SubTitle] [nvarchar](1000) NULL,
	[PublicModuleAndComponent] [nvarchar](256) NULL,
	[ExistingPageId] [int] NULL,
	[ViewsCount] [int] NOT NULL,
	[DislikesCount] [int] NOT NULL,
	[LikesCount] [int] NOT NULL,
	[RateAverage] [float] NOT NULL,
	[MenuData] [xml] NULL,
	[Invisible] [bit] NOT NULL,
	[FeaturedImage] [nvarchar](max) NULL,
	[DisableEn] [bit] NOT NULL,
	[DisableAr] [bit] NOT NULL,
 CONSTRAINT [PK_Menus] PRIMARY KEY CLUSTERED 
(
	[Id] ASC
)WITH (PAD_INDEX = OFF, STATISTICS_NORECOMPUTE = OFF, IGNORE_DUP_KEY = OFF, ALLOW_ROW_LOCKS = ON, ALLOW_PAGE_LOCKS = ON, OPTIMIZE_FOR_SEQUENTIAL_KEY = OFF) ON [PRIMARY]
) ON [PRIMARY] TEXTIMAGE_ON [PRIMARY]
GO
/****** Object:  View [dbo].[TranslatedMenus]    Script Date: 3/11/2024 8:29:35 AM ******/
SET ANSI_NULLS ON
GO
SET QUOTED_IDENTIFIER ON
GO
CREATE VIEW [dbo].[TranslatedMenus]
AS
SELECT        dbo.Menus.Id, dbo.GetTranslation(dbo.Translations.Fields, dbo.Menus.Name, N'Name') AS Name, dbo.GetTranslation(dbo.Translations.Fields, dbo.Menus.ShortName, N'ShortName') AS ShortName, 
                         dbo.GetTranslation(dbo.Translations.Fields, dbo.Menus.Title, N'Title') AS Title, dbo.GetTranslation(dbo.Translations.Fields, dbo.Menus.SubTitle, N'SubTitle') AS SubTitle, dbo.GetTranslation(dbo.Translations.Fields, 
                         dbo.Menus.Keywords, N'Keywords') AS Keywords, dbo.GetTranslation(dbo.Translations.Fields, dbo.Menus.Description, N'Description') AS Description, dbo.Menus.MenuType, dbo.Menus.Type, dbo.Menus.MenuResources, 
                         dbo.GetTranslation(dbo.Translations.Fields, dbo.Menus.IntroText, N'IntroText') AS IntroText, dbo.GetTranslation(dbo.Translations.Fields, dbo.Menus.PageText, N'PageText') AS PageText, 
                         dbo.GetTranslation(dbo.Translations.Fields, dbo.Menus.FeaturedImage, N'FeaturedImage') AS FeaturedImage, dbo.Menus.Link, dbo.Menus.LinkPattern, dbo.Menus.Target, dbo.Menus.Ordering, dbo.Menus.ClassName, 
                         dbo.Menus.Parent, dbo.Menus.SubLevel, dbo.Menus.DateCreated, dbo.Menus.CreatedBy, dbo.Menus.LastUpdated, dbo.Menus.LastUpdatedBy, dbo.Menus.IsHomepage, dbo.Menus.AuthorizedFunc, 
                         dbo.Translations.LanguageId, dbo.Menus.PublicModuleAndComponent, dbo.Menus.ExistingPageId, dbo.Menus.ViewsCount, dbo.Menus.DislikesCount, dbo.Menus.LikesCount, dbo.Menus.RateAverage, dbo.Menus.MenuData, 
                         dbo.Menus.Invisible, dbo.Menus.DisableEn , dbo.Menus.DisableAr
FROM            dbo.Menus LEFT OUTER JOIN
                         dbo.Translations ON dbo.Menus.Id = dbo.Translations.RefId AND dbo.Translations.RefType = 'N'
WHERE        (dbo.Menus.IsPublished = 1) AND (dbo.Menus.IsDeleted = 0)

GO
/****** Object:  Table [dbo].[StrategicPartners]    Script Date: 3/11/2024 8:29:35 AM ******/
SET ANSI_NULLS ON
GO
SET QUOTED_IDENTIFIER ON
GO
CREATE TABLE [dbo].[StrategicPartners](
	[Id] [uniqueidentifier] NOT NULL,
	[Name] [nvarchar](512) NOT NULL,
	[CountryId] [int] NOT NULL,
	[Link] [nvarchar](512) NULL,
	[Logo] [xml] NOT NULL,
	[IsDeleted] [bit] NOT NULL,
	[IsPublished] [bit] NOT NULL,
	[DateCreated] [datetime] NOT NULL,
	[CreatedBy] [nvarchar](128) NOT NULL,
	[LastUpdated] [datetime] NULL,
	[LastUpdatedBy] [nvarchar](128) NULL,
 CONSTRAINT [PK_StrategicPartners] PRIMARY KEY CLUSTERED 
(
	[Id] ASC
)WITH (PAD_INDEX = OFF, STATISTICS_NORECOMPUTE = OFF, IGNORE_DUP_KEY = OFF, ALLOW_ROW_LOCKS = ON, ALLOW_PAGE_LOCKS = ON, OPTIMIZE_FOR_SEQUENTIAL_KEY = OFF) ON [PRIMARY]
) ON [PRIMARY] TEXTIMAGE_ON [PRIMARY]
GO
/****** Object:  View [dbo].[TranslatedStrategicPartners]    Script Date: 3/11/2024 8:29:35 AM ******/
SET ANSI_NULLS ON
GO
SET QUOTED_IDENTIFIER ON
GO
CREATE VIEW [dbo].[TranslatedStrategicPartners]
AS
SELECT        dbo.StrategicPartners.Id, dbo.GetTranslation(dbo.Translations.Fields, dbo.StrategicPartners.Name, N'Name') AS Name, dbo.StrategicPartners.CountryId, dbo.StrategicPartners.Link, dbo.StrategicPartners.Logo, 
                         dbo.StrategicPartners.IsDeleted, dbo.StrategicPartners.IsPublished, dbo.StrategicPartners.DateCreated, dbo.StrategicPartners.CreatedBy, dbo.StrategicPartners.LastUpdated, dbo.StrategicPartners.LastUpdatedBy, 
                         dbo.Translations.LanguageId, dbo.GetTranslation(dbo.Translations.Fields, dbo.Constants.Name, N'CountryName') AS CountryName
FROM            dbo.StrategicPartners INNER JOIN
                         dbo.Constants ON dbo.Constants.Id = dbo.StrategicPartners.CountryId LEFT OUTER JOIN
                         dbo.Translations ON CONVERT(VARCHAR(50), dbo.StrategicPartners.Id) = dbo.Translations.RefId AND dbo.Translations.RefType = '6'
GO
/****** Object:  Table [dbo].[Announcements]    Script Date: 3/11/2024 8:29:35 AM ******/
SET ANSI_NULLS ON
GO
SET QUOTED_IDENTIFIER ON
GO
CREATE TABLE [dbo].[Announcements](
	[Id] [uniqueidentifier] NOT NULL,
	[Title] [nvarchar](512) NOT NULL,
	[Description] [nvarchar](max) NOT NULL,
	[FeaturedImage] [xml] NOT NULL,
	[IsDeleted] [bit] NOT NULL,
	[IsPublished] [bit] NOT NULL,
	[IsSticky] [bit] NOT NULL,
	[ItemDate] [datetime] NULL,
	[DateCreated] [datetime] NOT NULL,
	[CreatedBy] [nvarchar](128) NOT NULL,
	[LastUpdated] [datetime] NULL,
	[LastUpdatedBy] [nvarchar](128) NULL,
 CONSTRAINT [PK_Announcements] PRIMARY KEY CLUSTERED 
(
	[Id] ASC
)WITH (PAD_INDEX = OFF, STATISTICS_NORECOMPUTE = OFF, IGNORE_DUP_KEY = OFF, ALLOW_ROW_LOCKS = ON, ALLOW_PAGE_LOCKS = ON, OPTIMIZE_FOR_SEQUENTIAL_KEY = OFF) ON [PRIMARY]
) ON [PRIMARY] TEXTIMAGE_ON [PRIMARY]
GO
/****** Object:  View [dbo].[TranslatedAnnouncements]    Script Date: 3/11/2024 8:29:35 AM ******/
SET ANSI_NULLS ON
GO
SET QUOTED_IDENTIFIER ON
GO

CREATE VIEW [dbo].[TranslatedAnnouncements]
AS
SELECT        dbo.Announcements.Id, dbo.GetTranslation(dbo.Translations.Fields, dbo.Announcements.Title, N'Title') AS Title, dbo.GetTranslation(dbo.Translations.Fields, dbo.Announcements.Description, N'Description') AS Description, 
                         dbo.Announcements.FeaturedImage, dbo.Announcements.IsDeleted, dbo.Announcements.IsPublished, dbo.Announcements.DateCreated, dbo.Announcements.CreatedBy, dbo.Announcements.LastUpdated, 
                         dbo.Announcements.LastUpdatedBy, dbo.Translations.LanguageId, dbo.Announcements.ItemDate, dbo.Announcements.IsSticky
FROM            dbo.Announcements LEFT OUTER JOIN
                         dbo.Translations ON CONVERT(VARCHAR(50), dbo.Announcements.Id) = dbo.Translations.RefId AND dbo.Translations.RefType = '3'
GO
/****** Object:  Table [dbo].[Experts]    Script Date: 3/11/2024 8:29:35 AM ******/
SET ANSI_NULLS ON
GO
SET QUOTED_IDENTIFIER ON
GO
CREATE TABLE [dbo].[Experts](
	[Id] [int] IDENTITY(1,1) NOT NULL,
	[Name] [nvarchar](512) NOT NULL,
	[Description] [nvarchar](max) NOT NULL,
	[Photo] [xml] NULL,
	[Email] [nvarchar](200) NULL,
	[DateOfBirth] [datetime] NULL,
	[Languages] [nvarchar](512) NOT NULL,
	[Designation] [nvarchar](200) NULL,
	[Address] [nvarchar](1000) NULL,
	[CountryId] [int] NULL,
	[Gender] [char](1) NULL,
	[PlaceOfWork] [nvarchar](512) NULL,
	[PhoneNumber1] [nvarchar](50) NULL,
	[PhoneNumber2] [nvarchar](50) NULL,
	[Education] [nvarchar](512) NULL,
	[Type] [char](1) NOT NULL,
	[ShowInHomepage] [bit] NULL,
	[Slug] [nvarchar](512) NULL,
	[IsPublished] [bit] NOT NULL,
	[IsSticky] [bit] NULL,
	[IsDeleted] [bit] NOT NULL,
	[DateCreated] [datetime] NOT NULL,
	[CreatedBy] [nvarchar](128) NOT NULL,
	[LastUpdated] [datetime] NULL,
	[LastUpdatedBy] [nvarchar](128) NULL,
	[ViewsCount] [int] NOT NULL,
	[DislikesCount] [int] NOT NULL,
	[LikesCount] [int] NOT NULL,
	[RateAverage] [float] NOT NULL,
	[MetaKeywords] [nvarchar](1024) NULL,
	[MetaDescription] [nvarchar](max) NULL,
 CONSTRAINT [PK_Experts] PRIMARY KEY CLUSTERED 
(
	[Id] ASC
)WITH (PAD_INDEX = OFF, STATISTICS_NORECOMPUTE = OFF, IGNORE_DUP_KEY = OFF, ALLOW_ROW_LOCKS = ON, ALLOW_PAGE_LOCKS = ON, OPTIMIZE_FOR_SEQUENTIAL_KEY = OFF) ON [PRIMARY]
) ON [PRIMARY] TEXTIMAGE_ON [PRIMARY]
GO
/****** Object:  View [dbo].[TranslatedExperts]    Script Date: 3/11/2024 8:29:35 AM ******/
SET ANSI_NULLS ON
GO
SET QUOTED_IDENTIFIER ON
GO

CREATE VIEW [dbo].[TranslatedExperts]

AS

SELECT        dbo.Experts.Id, dbo.GetTranslation(dbo.Translations.Fields, dbo.Experts.Name, N'Name') AS Name, dbo.GetTranslation(dbo.Translations.Fields, dbo.Experts.Description, N'Description') AS Description, dbo.Experts.Email, 

                         dbo.Experts.DateOfBirth, dbo.Experts.Languages, dbo.GetTranslation(dbo.Translations.Fields, dbo.Experts.Designation, N'Designation') AS Designation, dbo.GetTranslation(dbo.Translations.Fields, dbo.Experts.Address, 

                         N'Address') AS Address, dbo.Experts.Gender, dbo.Experts.PlaceOfWork, dbo.Experts.PhoneNumber1, dbo.Experts.PhoneNumber2, dbo.GetTranslation(dbo.Translations.Fields, dbo.Experts.Education, N'Education') AS Education,

                          dbo.Experts.Type, dbo.Experts.IsPublished, dbo.Experts.IsDeleted, dbo.Experts.DateCreated, dbo.Experts.CreatedBy, dbo.Experts.LastUpdated, dbo.Experts.LastUpdatedBy, dbo.Experts.Slug, dbo.Experts.IsSticky, 

                         dbo.Experts.ShowInHomepage, dbo.Translations.LanguageId, dbo.Experts.CountryId, dbo.Constants.Name AS CountryName, dbo.Experts.ViewsCount, dbo.Experts.DislikesCount, dbo.Experts.LikesCount, 

                         dbo.Experts.RateAverage, dbo.Experts.MetaKeywords, dbo.Experts.MetaDescription, dbo.Experts.Photo

FROM            dbo.Experts LEFT OUTER JOIN

                         dbo.Translations ON CONVERT(VARCHAR(50), dbo.Experts.Id) = dbo.Translations.RefId AND dbo.Translations.RefType = '2' LEFT OUTER JOIN

                         dbo.Constants ON dbo.Experts.CountryId = dbo.Constants.Id

GO
/****** Object:  Table [dbo].[Languages]    Script Date: 3/11/2024 8:29:35 AM ******/
SET ANSI_NULLS ON
GO
SET QUOTED_IDENTIFIER ON
GO
CREATE TABLE [dbo].[Languages](
	[Id] [int] NOT NULL,
	[Name] [varchar](90) NOT NULL,
	[NativeName] [nvarchar](90) NOT NULL,
	[LanguageCode] [varchar](10) NOT NULL,
	[ContentCharSet] [varchar](50) NOT NULL,
	[IsRightToLeft] [bit] NOT NULL,
	[IsDefault] [bit] NOT NULL,
	[Localization] [nvarchar](max) NULL,
	[LanguageShortCode] [varchar](10) NULL,
 CONSTRAINT [PK_Languages] PRIMARY KEY CLUSTERED 
(
	[Id] ASC
)WITH (PAD_INDEX = OFF, STATISTICS_NORECOMPUTE = OFF, IGNORE_DUP_KEY = OFF, ALLOW_ROW_LOCKS = ON, ALLOW_PAGE_LOCKS = ON, OPTIMIZE_FOR_SEQUENTIAL_KEY = OFF) ON [PRIMARY]
) ON [PRIMARY] TEXTIMAGE_ON [PRIMARY]
GO
/****** Object:  View [dbo].[TranslatedLanguages]    Script Date: 3/11/2024 8:29:35 AM ******/
SET ANSI_NULLS ON
GO
SET QUOTED_IDENTIFIER ON
GO
CREATE VIEW [dbo].[TranslatedLanguages]
AS
SELECT        dbo.Languages.Id, dbo.Translations.Name, dbo.Languages.NativeName, dbo.Languages.LanguageCode, dbo.Languages.ContentCharSet, dbo.Languages.IsRightToLeft, dbo.Languages.IsDefault, dbo.Translations.LanguageId, 
                         dbo.Languages.Localization
FROM            dbo.Languages LEFT OUTER JOIN
                         dbo.Translations ON CONVERT(VARCHAR(50), dbo.Languages.Id) = dbo.Translations.RefId AND dbo.Translations.RefType = '('

GO
/****** Object:  Table [dbo].[Videos]    Script Date: 3/11/2024 8:29:35 AM ******/
SET ANSI_NULLS ON
GO
SET QUOTED_IDENTIFIER ON
GO
CREATE TABLE [dbo].[Videos](
	[Id] [uniqueidentifier] NOT NULL,
	[Title] [nvarchar](200) NOT NULL,
	[ItemDate] [datetime] NOT NULL,
	[Description] [nvarchar](max) NOT NULL,
	[FeaturedImage] [nvarchar](512) NULL,
	[YouTubeVideoId] [nvarchar](200) NULL,
	[UploadVideo] [nvarchar](512) NULL,
	[Tags] [nvarchar](1024) NULL,
	[ShowInHomepage] [bit] NOT NULL,
	[Slug] [nvarchar](512) NOT NULL,
	[IsSticky] [bit] NOT NULL,
	[LikesCount] [int] NOT NULL,
	[RateAverage] [float] NOT NULL,
	[ViewsCount] [int] NOT NULL,
	[ContentThemeIds] [nvarchar](512) NULL,
	[MetaKeywords] [nvarchar](1024) NULL,
	[MetaDescription] [nvarchar](max) NULL,
	[IsArchived] [bit] NOT NULL,
	[IsPublished] [bit] NOT NULL,
	[IsDeleted] [bit] NOT NULL,
	[DateCreated] [datetime] NOT NULL,
	[CreatedBy] [nvarchar](128) NOT NULL,
	[LastUpdated] [datetime] NOT NULL,
	[LastUpdatedBy] [nvarchar](128) NOT NULL,
	[DislikesCount] [int] NOT NULL,
	[CategoryId] [int] NULL,
	[IsCompressed] [bit] NOT NULL,
 CONSTRAINT [PK_Videos] PRIMARY KEY CLUSTERED 
(
	[Id] ASC
)WITH (PAD_INDEX = OFF, STATISTICS_NORECOMPUTE = OFF, IGNORE_DUP_KEY = OFF, ALLOW_ROW_LOCKS = ON, ALLOW_PAGE_LOCKS = ON, OPTIMIZE_FOR_SEQUENTIAL_KEY = OFF) ON [PRIMARY]
) ON [PRIMARY] TEXTIMAGE_ON [PRIMARY]
GO
/****** Object:  View [dbo].[TranslatedVideos]    Script Date: 3/11/2024 8:29:35 AM ******/
SET ANSI_NULLS ON
GO
SET QUOTED_IDENTIFIER ON
GO


CREATE VIEW [dbo].[TranslatedVideos]
AS
SELECT        dbo.Videos.Id, dbo.GetTranslation(dbo.Translations.Fields, dbo.Videos.Title, N'Title') AS Title, dbo.GetTranslation(dbo.Translations.Fields, dbo.Videos.Description, N'Description') AS Description, 
                         dbo.GetTranslation(dbo.Translations.Fields, dbo.Videos.MetaKeywords, N'MetaKeywords') AS MetaKeywords, dbo.GetTranslation(dbo.Translations.Fields, dbo.Videos.MetaDescription, N'MetaDescription') AS MetaDescription, 
                         dbo.Videos.ItemDate, dbo.GetTranslation(dbo.Translations.Fields, dbo.Videos.FeaturedImage, N'FeaturedImage') AS FeaturedImage, dbo.GetTranslation(dbo.Translations.Fields, dbo.Videos.YouTubeVideoId, N'YouTubeVideoId') AS YouTubeVideoId, 
						 dbo.GetTranslation(dbo.Translations.Fields, dbo.Videos.UploadVideo, N'UploadVideo') AS UploadVideo, dbo.Videos.Tags, dbo.Videos.IsArchived, dbo.Videos.IsPublished, dbo.Videos.IsDeleted, dbo.Videos.DateCreated, 
                         dbo.Videos.CreatedBy, dbo.Videos.LastUpdated, dbo.Videos.LastUpdatedBy, dbo.Translations.LanguageId, dbo.Videos.Slug, dbo.Videos.IsSticky, dbo.Videos.ShowInHomepage, dbo.Videos.ContentThemeIds, 
                         dbo.Videos.LikesCount, dbo.Videos.RateAverage, dbo.Videos.ViewsCount, dbo.Videos.DislikesCount, dbo.Videos.CategoryId, dbo.Videos.IsCompressed
FROM            dbo.Videos LEFT OUTER JOIN
                         dbo.Translations ON CONVERT(VARCHAR(50), dbo.Videos.Id) = dbo.Translations.RefId AND dbo.Translations.RefType = '4' LEFT OUTER JOIN
                         dbo.TranslatedCategories ON dbo.Videos.CategoryId = dbo.TranslatedCategories.Id AND dbo.Translations.LanguageId = dbo.TranslatedCategories.LanguageId
GO
/****** Object:  Table [dbo].[Banners]    Script Date: 3/11/2024 8:29:35 AM ******/
SET ANSI_NULLS ON
GO
SET QUOTED_IDENTIFIER ON
GO
CREATE TABLE [dbo].[Banners](
	[Id] [int] IDENTITY(1,1) NOT NULL,
	[Title] [nvarchar](500) NOT NULL,
	[Description] [nvarchar](max) NULL,
	[Link] [nvarchar](500) NULL,
	[Target] [char](1) NOT NULL,
	[OpenPopup] [bit] NOT NULL,
	[DisplayOrder] [int] NOT NULL,
	[BannerType] [char](1) NOT NULL,
	[Attachment] [xml] NOT NULL,
	[DateCreated] [datetime] NOT NULL,
	[CreatedBy] [nvarchar](128) NOT NULL,
	[LastUpdated] [datetime] NOT NULL,
	[LastUpdatedBy] [nvarchar](128) NOT NULL,
	[IsDeleted] [bit] NOT NULL,
 CONSTRAINT [PK_Banners] PRIMARY KEY CLUSTERED 
(
	[Id] ASC
)WITH (PAD_INDEX = OFF, STATISTICS_NORECOMPUTE = OFF, IGNORE_DUP_KEY = OFF, ALLOW_ROW_LOCKS = ON, ALLOW_PAGE_LOCKS = ON, OPTIMIZE_FOR_SEQUENTIAL_KEY = OFF) ON [PRIMARY]
) ON [PRIMARY] TEXTIMAGE_ON [PRIMARY]
GO
/****** Object:  View [dbo].[TranslatedBanners]    Script Date: 3/11/2024 8:29:35 AM ******/
SET ANSI_NULLS ON
GO
SET QUOTED_IDENTIFIER ON
GO
CREATE VIEW [dbo].[TranslatedBanners]
AS
SELECT        dbo.Banners.Id, ISNULL(dbo.Translations.Name, dbo.Banners.Title) AS Title, dbo.GetTranslation(dbo.Translations.Fields, dbo.Banners.Description, N'Description') AS Description, dbo.Banners.IsDeleted, dbo.Banners.Link, 
                         dbo.Banners.Target, dbo.Banners.OpenPopup, dbo.Banners.DisplayOrder, dbo.Banners.BannerType, dbo.Banners.Attachment, dbo.Translations.LanguageId
FROM            dbo.Banners LEFT OUTER JOIN
                         dbo.Translations ON dbo.Banners.Id = dbo.Translations.RefId AND dbo.Translations.RefType = 'B'
WHERE        (dbo.Banners.IsDeleted = 0)
GO
/****** Object:  Table [dbo].[Messages]    Script Date: 3/11/2024 8:29:35 AM ******/
SET ANSI_NULLS ON
GO
SET QUOTED_IDENTIFIER ON
GO
CREATE TABLE [dbo].[Messages](
	[Id] [int] IDENTITY(1,1) NOT NULL,
	[Title] [nvarchar](500) NOT NULL,
	[Body] [nvarchar](max) NULL,
	[Author] [nvarchar](500) NULL,
	[PublishDate] [datetime] NULL,
	[Attachments] [xml] NULL,
	[DateCreated] [datetime] NOT NULL,
	[CreatedBy] [nvarchar](128) NOT NULL,
	[LastUpdated] [datetime] NOT NULL,
	[LastUpdatedBy] [nvarchar](128) NOT NULL,
	[IsPublished] [bit] NOT NULL,
	[IsDeleted] [bit] NOT NULL,
 CONSTRAINT [PK_Messages] PRIMARY KEY CLUSTERED 
(
	[Id] ASC
)WITH (PAD_INDEX = OFF, STATISTICS_NORECOMPUTE = OFF, IGNORE_DUP_KEY = OFF, ALLOW_ROW_LOCKS = ON, ALLOW_PAGE_LOCKS = ON, OPTIMIZE_FOR_SEQUENTIAL_KEY = OFF) ON [PRIMARY]
) ON [PRIMARY] TEXTIMAGE_ON [PRIMARY]
GO
/****** Object:  View [dbo].[TranslatedMessages]    Script Date: 3/11/2024 8:29:35 AM ******/
SET ANSI_NULLS ON
GO
SET QUOTED_IDENTIFIER ON
GO
CREATE VIEW [dbo].[TranslatedMessages]
AS
SELECT        dbo.Messages.Id, ISNULL(dbo.Translations.Name, dbo.Messages.Title) AS Title, dbo.GetTranslation(dbo.Translations.Fields, dbo.Messages.Author, N'Author') AS Author, dbo.GetTranslation(dbo.Translations.Fields, 
                         dbo.Messages.Body, N'Body') AS Body, dbo.Messages.PublishDate, dbo.Messages.Attachments, dbo.Messages.IsPublished, dbo.Messages.IsDeleted, dbo.Translations.LanguageId
FROM            dbo.Messages LEFT OUTER JOIN
                         dbo.Translations ON dbo.Messages.Id = dbo.Translations.RefId AND dbo.Translations.RefType = 'S'
WHERE        (dbo.Messages.IsDeleted = 0)
GO
/****** Object:  Table [dbo].[Assets]    Script Date: 3/11/2024 8:29:35 AM ******/
SET ANSI_NULLS ON
GO
SET QUOTED_IDENTIFIER ON
GO
CREATE TABLE [dbo].[Assets](
	[Id] [int] IDENTITY(1,1) NOT NULL,
	[Title] [nvarchar](512) NOT NULL,
	[Description] [nvarchar](max) NULL,
	[Attachments] [xml] NOT NULL,
	[IsDeleted] [bit] NOT NULL,
	[IsPublished] [bit] NOT NULL,
	[DateCreated] [datetime] NOT NULL,
	[CreatedBy] [nvarchar](128) NOT NULL,
	[LastUpdated] [datetime] NULL,
	[LastUpdatedBy] [nvarchar](128) NULL,
 CONSTRAINT [PK_Assets] PRIMARY KEY CLUSTERED 
(
	[Id] ASC
)WITH (PAD_INDEX = OFF, STATISTICS_NORECOMPUTE = OFF, IGNORE_DUP_KEY = OFF, ALLOW_ROW_LOCKS = ON, ALLOW_PAGE_LOCKS = ON, OPTIMIZE_FOR_SEQUENTIAL_KEY = OFF) ON [PRIMARY]
) ON [PRIMARY] TEXTIMAGE_ON [PRIMARY]
GO
/****** Object:  View [dbo].[TranslatedAssets]    Script Date: 3/11/2024 8:29:35 AM ******/
SET ANSI_NULLS ON
GO
SET QUOTED_IDENTIFIER ON
GO


CREATE VIEW [dbo].[TranslatedAssets]
AS
SELECT        dbo.Assets.Id, dbo.GetTranslation(dbo.Translations.Fields, dbo.Assets.Title, N'Title') AS Title, dbo.GetTranslation(dbo.Translations.Fields, dbo.Assets.Description, N'Description') AS Description, dbo.Assets.Attachments, 
                         dbo.Assets.IsDeleted, dbo.Assets.IsPublished, dbo.Assets.DateCreated, dbo.Assets.CreatedBy, dbo.Assets.LastUpdated, dbo.Assets.LastUpdatedBy, dbo.Translations.LanguageId
FROM            dbo.Assets LEFT OUTER JOIN
                         dbo.Translations ON dbo.Assets.Id = dbo.Translations.RefId AND dbo.Translations.RefType = 'A'

GO
/****** Object:  Table [dbo].[ContentThemes]    Script Date: 3/11/2024 8:29:35 AM ******/
SET ANSI_NULLS ON
GO
SET QUOTED_IDENTIFIER ON
GO
CREATE TABLE [dbo].[ContentThemes](
	[Id] [int] IDENTITY(1,1) NOT NULL,
	[Name] [nvarchar](512) NOT NULL,
	[Description] [nvarchar](max) SPARSE  NULL,
	[IsDeleted] [bit] NOT NULL,
	[IsPublished] [bit] NOT NULL,
	[DateCreated] [datetime] NOT NULL,
	[CreatedBy] [nvarchar](128) NOT NULL,
	[LastUpdated] [datetime] NOT NULL,
	[LastUpdatedBy] [nvarchar](128) NOT NULL,
 CONSTRAINT [PK_ContentThemes] PRIMARY KEY CLUSTERED 
(
	[Id] ASC
)WITH (PAD_INDEX = OFF, STATISTICS_NORECOMPUTE = OFF, IGNORE_DUP_KEY = OFF, ALLOW_ROW_LOCKS = ON, ALLOW_PAGE_LOCKS = ON, OPTIMIZE_FOR_SEQUENTIAL_KEY = OFF) ON [PRIMARY]
) ON [PRIMARY] TEXTIMAGE_ON [PRIMARY]
GO
/****** Object:  View [dbo].[TranslatedContentThemes]    Script Date: 3/11/2024 8:29:35 AM ******/
SET ANSI_NULLS ON
GO
SET QUOTED_IDENTIFIER ON
GO
CREATE VIEW [dbo].[TranslatedContentThemes]
AS
SELECT        dbo.ContentThemes.Id, dbo.GetTranslation(dbo.Translations.Fields, dbo.ContentThemes.Name, N'Name') AS Name, dbo.GetTranslation(dbo.Translations.Fields, dbo.ContentThemes.Description, N'Description') AS Description, 
                         dbo.ContentThemes.IsDeleted, dbo.ContentThemes.IsPublished, dbo.ContentThemes.DateCreated, dbo.ContentThemes.CreatedBy, dbo.ContentThemes.LastUpdated, dbo.ContentThemes.LastUpdatedBy, 
                         dbo.Translations.LanguageId
FROM            dbo.ContentThemes LEFT OUTER JOIN
                         dbo.Translations ON CONVERT(VARCHAR(50), dbo.ContentThemes.Id) = dbo.Translations.RefId AND dbo.Translations.RefType = '/'
GO
/****** Object:  Table [dbo].[NewsExperts]    Script Date: 3/11/2024 8:29:35 AM ******/
SET ANSI_NULLS ON
GO
SET QUOTED_IDENTIFIER ON
GO
CREATE TABLE [dbo].[NewsExperts](
	[Id] [uniqueidentifier] NOT NULL,
	[ExpertId] [int] NOT NULL,
	[NewsId] [uniqueidentifier] NOT NULL,
	[ExpertCategoryId] [int] NOT NULL,
 CONSTRAINT [PK_NewsExperts] PRIMARY KEY CLUSTERED 
(
	[Id] ASC
)WITH (PAD_INDEX = OFF, STATISTICS_NORECOMPUTE = OFF, IGNORE_DUP_KEY = OFF, ALLOW_ROW_LOCKS = ON, ALLOW_PAGE_LOCKS = ON, OPTIMIZE_FOR_SEQUENTIAL_KEY = OFF) ON [PRIMARY]
) ON [PRIMARY]
GO
/****** Object:  Table [dbo].[News]    Script Date: 3/11/2024 8:29:35 AM ******/
SET ANSI_NULLS ON
GO
SET QUOTED_IDENTIFIER ON
GO
CREATE TABLE [dbo].[News](
	[Id] [uniqueidentifier] NOT NULL,
	[Title] [nvarchar](512) NOT NULL,
	[Description] [nvarchar](max) NOT NULL,
	[ItemDate] [datetime] NOT NULL,
	[Images] [xml] NULL,
	[CategoryId] [int] SPARSE  NULL,
	[FeaturedImage] [xml] NOT NULL,
	[IsArchived] [bit] NOT NULL,
	[IsSticky] [bit] NOT NULL,
	[ShowInHomepage] [bit] NOT NULL,
	[Slug] [nvarchar](512) NOT NULL,
	[ContentThemeIds] [nvarchar](max) SPARSE  NULL,
	[MetaKeywords] [nvarchar](1024) NULL,
	[MetaDescription] [nvarchar](max) NULL,
	[ViewsCount] [int] NOT NULL,
	[LikesCount] [int] NOT NULL,
	[RateAverage] [float] NOT NULL,
	[IsPublished] [bit] NOT NULL,
	[IsDeleted] [bit] NOT NULL,
	[CreatedBy] [nvarchar](128) NOT NULL,
	[DateCreated] [datetime] NOT NULL,
	[LastUpdated] [datetime] NULL,
	[LastUpdatedBy] [nvarchar](128) NULL,
	[DislikesCount] [int] NOT NULL,
 CONSTRAINT [PK_News_1] PRIMARY KEY CLUSTERED 
(
	[Id] ASC
)WITH (PAD_INDEX = OFF, STATISTICS_NORECOMPUTE = OFF, IGNORE_DUP_KEY = OFF, ALLOW_ROW_LOCKS = ON, ALLOW_PAGE_LOCKS = ON, OPTIMIZE_FOR_SEQUENTIAL_KEY = OFF) ON [PRIMARY]
) ON [PRIMARY] TEXTIMAGE_ON [PRIMARY]
GO
/****** Object:  View [dbo].[TranslatedNews]    Script Date: 3/11/2024 8:29:35 AM ******/
SET ANSI_NULLS ON
GO
SET QUOTED_IDENTIFIER ON
GO
CREATE VIEW [dbo].[TranslatedNews]
AS
SELECT        dbo.News.Id, dbo.GetTranslation(dbo.Translations.Fields, dbo.News.Title, N'Title') AS Title, dbo.GetTranslation(dbo.Translations.Fields, dbo.News.Description, N'Description') AS Description, 
				dbo.GetTranslation(dbo.Translations.Fields, dbo.News.MetaKeywords, N'MetaKeywords') AS MetaKeywords, dbo.GetTranslation(dbo.Translations.Fields, dbo.News.MetaDescription, N'MetaDescription') AS MetaDescription,
				dbo.News.ItemDate, dbo.News.CategoryId, dbo.TranslatedCategories.Name AS CategoryName,
                         dbo.News.Images, dbo.News.FeaturedImage, dbo.News.IsArchived, dbo.News.IsSticky, dbo.News.Slug, dbo.News.IsPublished, dbo.News.IsDeleted, dbo.News.DateCreated, dbo.News.CreatedBy, dbo.News.LastUpdated, 
                         dbo.News.LastUpdatedBy, dbo.Translations.LanguageId, dbo.News.ShowInHomepage,dbo.News.ContentThemeIds, dbo.News.ViewsCount,  dbo.News.LikesCount,  dbo.News.DislikesCount, dbo.News.RateAverage,
                             (SELECT        (',' + STUFF
                                                             ((SELECT        ',' + cast(pe.ExpertId AS nvarchar(50))
                                                                 FROM            dbo.NewsExperts pe
                                                                 WHERE        pe.NewsId = dbo.News.Id FOR XML PATH('')), 1, 1, '') + ',')) AS ExpertsIds
FROM            dbo.News LEFT OUTER JOIN
                         dbo.Translations ON CONVERT(VARCHAR(50), dbo.News.Id) = dbo.Translations.RefId AND dbo.Translations.RefType = '7'
                         LEFT OUTER JOIN dbo.TranslatedCategories ON dbo.News.CategoryId = dbo.TranslatedCategories.Id AND dbo.Translations.LanguageId = dbo.TranslatedCategories.LanguageId
GO
/****** Object:  Table [dbo].[EventExperts]    Script Date: 3/11/2024 8:29:35 AM ******/
SET ANSI_NULLS ON
GO
SET QUOTED_IDENTIFIER ON
GO
CREATE TABLE [dbo].[EventExperts](
	[Id] [uniqueidentifier] NOT NULL,
	[ExpertId] [int] NOT NULL,
	[EventId] [uniqueidentifier] NOT NULL,
	[ExpertCategoryId] [int] NOT NULL,
 CONSTRAINT [PK_EventExperts] PRIMARY KEY CLUSTERED 
(
	[Id] ASC
)WITH (PAD_INDEX = OFF, STATISTICS_NORECOMPUTE = OFF, IGNORE_DUP_KEY = OFF, ALLOW_ROW_LOCKS = ON, ALLOW_PAGE_LOCKS = ON, OPTIMIZE_FOR_SEQUENTIAL_KEY = OFF) ON [PRIMARY]
) ON [PRIMARY]
GO
/****** Object:  Table [dbo].[Events]    Script Date: 3/11/2024 8:29:35 AM ******/
SET ANSI_NULLS ON
GO
SET QUOTED_IDENTIFIER ON
GO
CREATE TABLE [dbo].[Events](
	[Id] [uniqueidentifier] NOT NULL,
	[Title] [nvarchar](512) NOT NULL,
	[Description] [nvarchar](max) NOT NULL,
	[EventStartDate] [datetime] NULL,
	[EventEndDate] [datetime] NULL,
	[YoutubeVideoId] [nvarchar](512) NULL,
	[Videos] [xml] NULL,
	[Attachments] [xml] NULL,
	[Images] [xml] NULL,
	[CategoryId] [int] SPARSE  NULL,
	[IsRegistrationRequired] [bit] NOT NULL,
	[MaxAttendancesCount] [int] NOT NULL,
	[IsOpenForRegistration] [bit] NOT NULL,
	[ContentThemeIds] [nvarchar](max) SPARSE  NULL,
	[MetaKeywords] [nvarchar](1024) SPARSE  NULL,
	[MetaDescription] [nvarchar](max) SPARSE  NULL,
	[ViewsCount] [int] NOT NULL,
	[LikesCount] [int] NOT NULL,
	[RateAverage] [float] NOT NULL,
	[Slug] [nvarchar](1024) NOT NULL,
	[IsSticky] [bit] NOT NULL,
	[ShowInHomepage] [bit] NOT NULL,
	[IsDeleted] [bit] NOT NULL,
	[IsPublished] [bit] NOT NULL,
	[DateCreated] [datetime] NOT NULL,
	[CreatedBy] [nvarchar](128) NOT NULL,
	[LastUpdated] [datetime] NULL,
	[LastUpdatedBy] [nvarchar](128) NULL,
	[DislikesCount] [int] NOT NULL,
	[Venue] [nvarchar](1024) NULL,
	[ApprovalRequired] [bit] NOT NULL,
 CONSTRAINT [PK_Events] PRIMARY KEY CLUSTERED 
(
	[Id] ASC
)WITH (PAD_INDEX = OFF, STATISTICS_NORECOMPUTE = OFF, IGNORE_DUP_KEY = OFF, ALLOW_ROW_LOCKS = ON, ALLOW_PAGE_LOCKS = ON, OPTIMIZE_FOR_SEQUENTIAL_KEY = OFF) ON [PRIMARY]
) ON [PRIMARY] TEXTIMAGE_ON [PRIMARY]
GO
/****** Object:  View [dbo].[TranslatedEvents]    Script Date: 3/11/2024 8:29:35 AM ******/
SET ANSI_NULLS ON
GO
SET QUOTED_IDENTIFIER ON
GO

CREATE VIEW [dbo].[TranslatedEvents]
AS
SELECT        dbo.Events.Id, dbo.GetTranslation(dbo.Translations.Fields, dbo.Events.Title, N'Title') AS Title, dbo.GetTranslation(dbo.Translations.Fields, dbo.Events.Description, N'Description') AS Description, 
                         dbo.GetTranslation(dbo.Translations.Fields, dbo.Events.MetaKeywords, N'MetaKeywords') AS MetaKeywords, dbo.GetTranslation(dbo.Translations.Fields, dbo.Events.MetaDescription, N'MetaDescription') AS MetaDescription, 
                         dbo.Events.EventStartDate, dbo.Events.CategoryId, dbo.TranslatedCategories.Name AS CategoryName, dbo.Events.EventEndDate, dbo.Events.YoutubeVideoId, dbo.Events.Videos, dbo.Events.Attachments, dbo.Events.Images, 
                         dbo.Events.IsRegistrationRequired, dbo.Events.MaxAttendancesCount, dbo.Events.IsOpenForRegistration, dbo.Events.Slug, dbo.Events.IsPublished, dbo.Events.IsDeleted, dbo.Events.DateCreated, dbo.Events.CreatedBy, 
                         dbo.Events.LastUpdated, dbo.Events.LastUpdatedBy, dbo.Events.IsSticky, dbo.Events.ContentThemeIds, dbo.Events.ShowInHomepage, dbo.Translations.LanguageId, dbo.Events.ViewsCount, dbo.Events.LikesCount, 
                         dbo.Events.DislikesCount, dbo.Events.RateAverage, dbo.GetTranslation(dbo.Translations.Fields, dbo.Events.Venue, N'Venue') AS Venue,dbo.Events.ApprovalRequired,
                             (SELECT        (',' + STUFF
                                                             ((SELECT        ',' + cast(pe.ExpertId AS nvarchar(50))
                                                                 FROM            dbo.EventExperts pe
                                                                 WHERE        pe.EventId = dbo.Events.Id FOR XML PATH('')), 1, 1, '') + ',')) AS ExpertsIds
FROM            dbo.Events LEFT OUTER JOIN
                         dbo.Translations ON CONVERT(VARCHAR(50), dbo.Events.Id) = dbo.Translations.RefId AND dbo.Translations.RefType = '8' LEFT OUTER JOIN
                         dbo.TranslatedCategories ON dbo.Events.CategoryId = dbo.TranslatedCategories.Id AND dbo.Translations.LanguageId = dbo.TranslatedCategories.LanguageId

GO
/****** Object:  Table [dbo].[ProductOptions]    Script Date: 3/11/2024 8:29:35 AM ******/
SET ANSI_NULLS ON
GO
SET QUOTED_IDENTIFIER ON
GO
CREATE TABLE [dbo].[ProductOptions](
	[Id] [uniqueidentifier] NOT NULL,
	[ProductId] [uniqueidentifier] NOT NULL,
	[ProductType] [char](1) NOT NULL,
	[ProductSpecification] [nvarchar](512) NULL,
	[LanguageId] [int] NULL,
	[Weight] [float] NULL,
	[Length] [float] NULL,
	[Width] [float] NULL,
	[Height] [float] NULL,
	[Attachment] [xml] NULL,
	[DeliveryMethod] [char](1) NULL,
	[DownloadLimit] [int] NULL,
	[DownloadExpiryDays] [int] NULL,
	[Price] [float] NOT NULL,
	[SalePrice] [float] NULL,
	[FeaturedImage] [xml] NULL,
	[StockStatus] [char](1) NOT NULL,
	[Stock] [int] NULL,
	[SalesCount] [int] NOT NULL,
	[PurchasesCount] [int] NOT NULL,
	[IsDeleted] [bit] NOT NULL,
	[IsPublished] [bit] NOT NULL,
	[IsTaxIncluded] [bit] NOT NULL,
	[IsLimitedStock] [bit] NOT NULL,
	[ISBN] [nvarchar](256) NULL,
	[PagesCount] [int] NULL,
	[Edition] [int] NULL,
	[CurrentEditionYear] [int] NULL,
	[FirstEditionYear] [int] NULL,
	[PublishDate] [datetime] NULL,
	[PreviewPages] [int] NULL,
	[IsAwarded] [bit] NOT NULL,
	[AwardYear] [int] NULL,
 CONSTRAINT [PK_PublicationDetails] PRIMARY KEY CLUSTERED 
(
	[Id] ASC
)WITH (PAD_INDEX = OFF, STATISTICS_NORECOMPUTE = OFF, IGNORE_DUP_KEY = OFF, ALLOW_ROW_LOCKS = ON, ALLOW_PAGE_LOCKS = ON, OPTIMIZE_FOR_SEQUENTIAL_KEY = OFF) ON [PRIMARY]
) ON [PRIMARY] TEXTIMAGE_ON [PRIMARY]
GO
/****** Object:  Table [dbo].[ProductExperts]    Script Date: 3/11/2024 8:29:35 AM ******/
SET ANSI_NULLS ON
GO
SET QUOTED_IDENTIFIER ON
GO
CREATE TABLE [dbo].[ProductExperts](
	[Id] [uniqueidentifier] NOT NULL,
	[ExpertId] [int] NOT NULL,
	[ProductId] [uniqueidentifier] NOT NULL,
	[ExpertType] [char](1) NOT NULL,
 CONSTRAINT [PK_ProductExperts] PRIMARY KEY CLUSTERED 
(
	[Id] ASC
)WITH (PAD_INDEX = OFF, STATISTICS_NORECOMPUTE = OFF, IGNORE_DUP_KEY = OFF, ALLOW_ROW_LOCKS = ON, ALLOW_PAGE_LOCKS = ON, OPTIMIZE_FOR_SEQUENTIAL_KEY = OFF) ON [PRIMARY]
) ON [PRIMARY]
GO
/****** Object:  Table [dbo].[Products]    Script Date: 3/11/2024 8:29:35 AM ******/
SET ANSI_NULLS ON
GO
SET QUOTED_IDENTIFIER ON
GO
CREATE TABLE [dbo].[Products](
	[Id] [uniqueidentifier] NOT NULL,
	[Title] [nvarchar](512) NOT NULL,
	[Description] [nvarchar](max) NOT NULL,
	[FullText] [nvarchar](max) NULL,
	[Type] [char](1) NOT NULL,
	[CategoryId] [int] NOT NULL,
	[SubCategoryId] [int] NULL,
	[IsBestSeller] [bit] NOT NULL,
	[ContentThemeIds] [nvarchar](max) SPARSE  NULL,
	[MetaKeywords] [nvarchar](1024) SPARSE  NULL,
	[MetaDescription] [nvarchar](max) SPARSE  NULL,
	[ViewsCount] [int] NOT NULL,
	[LikesCount] [int] NOT NULL,
	[RateAverage] [float] NULL,
	[Slug] [nvarchar](512) NOT NULL,
	[IsSticky] [bit] NOT NULL,
	[ShowInHomepage] [bit] NOT NULL,
	[IsDeleted] [bit] NOT NULL,
	[IsPublished] [bit] NOT NULL,
	[DateCreated] [datetime] NOT NULL,
	[CreatedBy] [nvarchar](128) NOT NULL,
	[LastUpdated] [datetime] NULL,
	[LastUpdatedBy] [nvarchar](128) NULL,
	[DislikesCount] [int] NOT NULL,
	[PublishDate] [datetime] NULL,
 CONSTRAINT [PK_Publications] PRIMARY KEY CLUSTERED 
(
	[Id] ASC
)WITH (PAD_INDEX = OFF, STATISTICS_NORECOMPUTE = OFF, IGNORE_DUP_KEY = OFF, ALLOW_ROW_LOCKS = ON, ALLOW_PAGE_LOCKS = ON, OPTIMIZE_FOR_SEQUENTIAL_KEY = OFF) ON [PRIMARY]
) ON [PRIMARY] TEXTIMAGE_ON [PRIMARY]
GO
/****** Object:  View [dbo].[TranslatedProducts]    Script Date: 3/11/2024 8:29:35 AM ******/
SET ANSI_NULLS ON
GO
SET QUOTED_IDENTIFIER ON
GO

CREATE VIEW [dbo].[TranslatedProducts]
AS
SELECT        dbo.Products.Id, dbo.GetTranslation(dbo.Translations.Fields, dbo.Products.Title, N'Title') AS Title, dbo.GetTranslation(dbo.Translations.Fields, dbo.Products.Description, N'Description') AS Description, 
                         dbo.GetTranslation(dbo.Translations.Fields, dbo.Products.FullText, N'FullText') AS FullText, dbo.GetTranslation(dbo.Translations.Fields, dbo.Products.MetaKeywords, N'MetaKeywords') AS MetaKeywords, 
                         dbo.GetTranslation(dbo.Translations.Fields, dbo.Products.MetaDescription, N'MetaDescription') AS MetaDescription, dbo.Products.Type, dbo.Products.ContentThemeIds, dbo.Products.CategoryId, 
                         MainCategories.Name AS CategoryName, dbo.Products.SubCategoryId, SubCategories.Name AS SubCategoryName, dbo.Products.IsBestSeller, dbo.Products.IsPublished, dbo.Products.IsDeleted, dbo.Products.DateCreated, 
                         dbo.Products.CreatedBy, dbo.Products.LastUpdated, dbo.Products.LastUpdatedBy, dbo.Translations.LanguageId, dbo.Products.Slug, dbo.Products.IsSticky, dbo.Products.ShowInHomepage, dbo.Products.ViewsCount, 
                         dbo.Products.LikesCount, dbo.Products.DislikesCount, dbo.Products.RateAverage, dbo.Products.PublishDate,
                             (SELECT        (',' + STUFF
                                                             ((SELECT        ',' + cast(pe.ExpertId AS nvarchar(50))
                                                                 FROM            dbo.ProductExperts pe
                                                                 WHERE        pe.ProductId = dbo.Products.Id AND pe.ExpertType = 'E' FOR XML PATH('')), 1, 1, '') + ',')) AS EditorsIds,
                             (SELECT        (',' + STUFF
                                                             ((SELECT        ',' + cast(pe.ExpertId AS nvarchar(50))
                                                                 FROM            dbo.ProductExperts pe
                                                                 WHERE        pe.ProductId = dbo.Products.Id AND pe.ExpertType = 'A' FOR XML PATH('')), 1, 1, '') + ',')) AS AuthorsIds,
                             (SELECT        (',' + STUFF
                                                             ((SELECT        ',' + cast(po.ISBN AS nvarchar(50))
                                                                 FROM            dbo.ProductOptions po
                                                                 WHERE        po.ProductId = dbo.Products.Id FOR XML PATH('')), 1, 1, '') + ',')) AS ISBNs,
                             (SELECT        (',' + STUFF
                                                             ((SELECT        ',' + cast(po.LanguageId AS nvarchar(50))
                                                                 FROM            dbo.ProductOptions po
                                                                 WHERE        po.ProductId = dbo.Products.Id FOR XML PATH('')), 1, 1, '') + ',')) AS Languages,
                             (SELECT		MAX(po.CurrentEditionYear)
                                                                 FROM            dbo.ProductOptions po
                                                                 WHERE        po.ProductId = dbo.Products.Id ) AS CurrentEditionYear
FROM            dbo.Products LEFT OUTER JOIN
                         dbo.Translations ON CONVERT(VARCHAR(50), dbo.Products.Id) = dbo.Translations.RefId AND dbo.Translations.RefType = '>' LEFT OUTER JOIN
                         dbo.TranslatedCategories MainCategories ON dbo.Products.CategoryId = MainCategories.Id AND dbo.Translations.LanguageId = MainCategories.LanguageId LEFT OUTER JOIN
                         dbo.TranslatedCategories SubCategories ON dbo.Products.SubCategoryId = SubCategories.Id AND dbo.Translations.LanguageId = SubCategories.LanguageId


GO
/****** Object:  Table [dbo].[Projects]    Script Date: 3/11/2024 8:29:35 AM ******/
SET ANSI_NULLS ON
GO
SET QUOTED_IDENTIFIER ON
GO
CREATE TABLE [dbo].[Projects](
	[Id] [uniqueidentifier] NOT NULL,
	[Title] [nvarchar](512) NOT NULL,
	[Description] [nvarchar](max) NOT NULL,
	[FeaturedImage] [xml] NOT NULL,
	[URL] [nvarchar](512) NULL,
	[IsSticky] [bit] NULL,
	[ShowInHomepage] [bit] NULL,
	[Slug] [nvarchar](512) NULL,
	[IsDeleted] [bit] NOT NULL,
	[IsPublished] [bit] NOT NULL,
	[DateCreated] [datetime] NOT NULL,
	[CreatedBy] [nvarchar](128) NOT NULL,
	[LastUpdated] [datetime] NULL,
	[LastUpdatedBy] [nvarchar](128) NULL,
	[ViewsCount] [int] NOT NULL,
	[DislikesCount] [int] NOT NULL,
	[LikesCount] [int] NOT NULL,
	[MetaKeywords] [nvarchar](1024) NULL,
	[MetaDescription] [nvarchar](max) NULL,
	[RateAverage] [float] NOT NULL,
 CONSTRAINT [PK_Projects] PRIMARY KEY CLUSTERED 
(
	[Id] ASC
)WITH (PAD_INDEX = OFF, STATISTICS_NORECOMPUTE = OFF, IGNORE_DUP_KEY = OFF, ALLOW_ROW_LOCKS = ON, ALLOW_PAGE_LOCKS = ON, OPTIMIZE_FOR_SEQUENTIAL_KEY = OFF) ON [PRIMARY]
) ON [PRIMARY] TEXTIMAGE_ON [PRIMARY]
GO
/****** Object:  View [dbo].[TranslatedProjects]    Script Date: 3/11/2024 8:29:35 AM ******/
SET ANSI_NULLS ON
GO
SET QUOTED_IDENTIFIER ON
GO
CREATE VIEW [dbo].[TranslatedProjects]
AS
SELECT        dbo.Projects.Id, dbo.GetTranslation(dbo.Translations.Fields, dbo.Projects.Title, N'Title') AS Title, dbo.GetTranslation(dbo.Translations.Fields, dbo.Projects.Description, N'Description') AS Description, 
                         dbo.GetTranslation(dbo.Translations.Fields, dbo.Projects.MetaKeywords, N'MetaKeywords') AS MetaKeywords, dbo.GetTranslation(dbo.Translations.Fields, dbo.Projects.MetaDescription, N'MetaDescription') 
                         AS MetaDescription, dbo.Projects.FeaturedImage, dbo.Projects.URL, dbo.Projects.IsDeleted, dbo.Projects.IsPublished, dbo.Projects.DateCreated, dbo.Projects.CreatedBy, dbo.Projects.LastUpdated, dbo.Projects.LastUpdatedBy, 
                         dbo.Translations.LanguageId, dbo.Projects.IsSticky, dbo.Projects.ShowInHomepage, dbo.Projects.Slug, dbo.Projects.ViewsCount, dbo.Projects.DislikesCount, dbo.Projects.LikesCount, dbo.Projects.RateAverage
FROM            dbo.Projects LEFT OUTER JOIN
                         dbo.Translations ON CONVERT(VARCHAR(50), dbo.Projects.Id) = dbo.Translations.RefId AND dbo.Translations.RefType = '9'
GO
/****** Object:  Table [dbo].[FAQs]    Script Date: 3/11/2024 8:29:35 AM ******/
SET ANSI_NULLS ON
GO
SET QUOTED_IDENTIFIER ON
GO
CREATE TABLE [dbo].[FAQs](
	[Id] [uniqueidentifier] NOT NULL,
	[Question] [nvarchar](512) NOT NULL,
	[Answer] [nvarchar](max) NOT NULL,
	[MetaKeywords] [nvarchar](1024) NULL,
	[MetaDescription] [nvarchar](max) NULL,
	[ViewsCount] [int] NOT NULL,
	[LikesCount] [int] NOT NULL,
	[RateAverage] [float] NOT NULL,
	[IsPublished] [bit] NOT NULL,
	[IsDeleted] [bit] NOT NULL,
	[DateCreated] [datetime] NOT NULL,
	[CreatedBy] [nvarchar](128) NOT NULL,
	[LastUpdated] [datetime] NULL,
	[LastUpdatedBy] [nvarchar](128) NULL,
	[DisplayOrder] [int] NOT NULL,
	[DislikesCount] [int] NOT NULL,
	[CategoryId] [int] NULL,
 CONSTRAINT [PK_FAQs] PRIMARY KEY CLUSTERED 
(
	[Id] ASC
)WITH (PAD_INDEX = OFF, STATISTICS_NORECOMPUTE = OFF, IGNORE_DUP_KEY = OFF, ALLOW_ROW_LOCKS = ON, ALLOW_PAGE_LOCKS = ON, OPTIMIZE_FOR_SEQUENTIAL_KEY = OFF) ON [PRIMARY]
) ON [PRIMARY] TEXTIMAGE_ON [PRIMARY]
GO
/****** Object:  View [dbo].[TranslatedFAQs]    Script Date: 3/11/2024 8:29:35 AM ******/
SET ANSI_NULLS ON
GO
SET QUOTED_IDENTIFIER ON
GO

CREATE VIEW [dbo].[TranslatedFAQs]
AS
SELECT        dbo.FAQs.Id, dbo.GetTranslation(dbo.Translations.Fields, dbo.FAQs.Question, N'Question') AS Question, dbo.GetTranslation(dbo.Translations.Fields, dbo.FAQs.Answer, N'Answer') AS Answer, dbo.FAQs.IsPublished, 

                         dbo.FAQs.DisplayOrder, dbo.FAQs.IsDeleted, dbo.FAQs.DateCreated, dbo.FAQs.CreatedBy, dbo.FAQs.LastUpdated, dbo.FAQs.LastUpdatedBy, dbo.Translations.LanguageId, dbo.FAQs.CategoryId
FROM            dbo.FAQs LEFT OUTER JOIN

                         dbo.Translations ON CONVERT(VARCHAR(50), dbo.FAQs.Id) = dbo.Translations.RefId AND dbo.Translations.RefType = '-'
GO
/****** Object:  Table [dbo].[SSR]    Script Date: 3/11/2024 8:29:35 AM ******/
SET ANSI_NULLS ON
GO
SET QUOTED_IDENTIFIER ON
GO
CREATE TABLE [dbo].[SSR](
	[Id] [uniqueidentifier] NOT NULL,
	[Title] [nvarchar](512) NOT NULL,
	[Description] [nvarchar](max) NULL,
	[FullText] [nvarchar](max) NULL,
	[MetaKeywords] [nvarchar](1024) NULL,
	[MetaDescription] [nvarchar](max) NULL,
	[ItemDate] [datetime] NOT NULL,
	[CategoryId] [int] NOT NULL,
	[Type] [int] NOT NULL,
	[ExpertId] [int] NULL,
	[Slug] [nvarchar](512) NOT NULL,
	[ContentThemeIds] [nvarchar](max) SPARSE  NULL,
	[TopicId] [int] NULL,
	[CountryId] [int] NULL,
	[AudioAttachments] [nvarchar](max) NULL,
	[VideoAttachments] [nvarchar](max) NULL,
	[Attachments] [nvarchar](max) NULL,
	[FeaturedImage] [nvarchar](512) NOT NULL,
	[ViewsCount] [int] NOT NULL,
	[LikesCount] [int] NOT NULL,
	[DislikesCount] [int] NOT NULL,
	[RateAverage] [float] NOT NULL,
	[IsSticky] [bit] NOT NULL,
	[IsPublished] [bit] NOT NULL,
	[IsDeleted] [bit] NOT NULL,
	[CreatedBy] [nvarchar](128) NOT NULL,
	[DateCreated] [datetime] NOT NULL,
	[LastUpdated] [datetime] NULL,
	[LastUpdatedBy] [nvarchar](128) NULL,
 CONSTRAINT [PK_SSR] PRIMARY KEY CLUSTERED 
(
	[Id] ASC
)WITH (PAD_INDEX = OFF, STATISTICS_NORECOMPUTE = OFF, IGNORE_DUP_KEY = OFF, ALLOW_ROW_LOCKS = ON, ALLOW_PAGE_LOCKS = ON, OPTIMIZE_FOR_SEQUENTIAL_KEY = OFF) ON [PRIMARY]
) ON [PRIMARY] TEXTIMAGE_ON [PRIMARY]
GO
/****** Object:  View [dbo].[TranslatedSSR]    Script Date: 3/11/2024 8:29:35 AM ******/
SET ANSI_NULLS ON
GO
SET QUOTED_IDENTIFIER ON
GO



CREATE VIEW [dbo].[TranslatedSSR]
AS
SELECT        dbo.SSR.Id, dbo.GetTranslation(dbo.Translations.Fields, dbo.SSR.Title, N'Title') AS Title, dbo.GetTranslation(dbo.Translations.Fields, dbo.SSR.Description, N'Description') AS Description, 
                         dbo.GetTranslation(dbo.Translations.Fields, dbo.SSR.FullText, N'FullText') AS FullText, dbo.GetTranslation(dbo.Translations.Fields, dbo.SSR.MetaKeywords, N'MetaKeywords') AS MetaKeywords, 
                         dbo.GetTranslation(dbo.Translations.Fields, dbo.SSR.MetaDescription, N'MetaDescription') AS MetaDescription, dbo.GetTranslation(dbo.Translations.Fields, dbo.SSR.FeaturedImage, N'FeaturedImage') AS FeaturedImage, 
                         dbo.GetTranslation(dbo.Translations.Fields, dbo.SSR.AudioAttachments, N'AudioAttachments') AS AudioAttachments, dbo.GetTranslation(dbo.Translations.Fields, dbo.SSR.VideoAttachments, N'VideoAttachments') 
                         AS VideoAttachments, dbo.GetTranslation(dbo.Translations.Fields, dbo.SSR.Attachments, N'Attachments') AS Attachments, dbo.SSR.Type, dbo.TranslatedConstants.Name AS TypeName, dbo.SSR.ExpertId, dbo.SSR.CategoryId, 
                         dbo.SSR.Slug, dbo.TranslatedCategories.Name AS CategoryName, dbo.SSR.ContentThemeIds, dbo.SSR.TopicId, dbo.SSR.CountryId, dbo.SSR.ViewsCount, dbo.SSR.LikesCount, dbo.SSR.DislikesCount, dbo.SSR.RateAverage, 
                         dbo.SSR.IsSticky, dbo.SSR.IsPublished, dbo.SSR.IsDeleted, dbo.SSR.CreatedBy, dbo.SSR.DateCreated, dbo.SSR.LastUpdated, dbo.SSR.LastUpdatedBy, dbo.Translations.LanguageId, dbo.SSR.ItemDate
FROM            dbo.SSR LEFT OUTER JOIN
                         dbo.Translations ON CONVERT(VARCHAR(50), dbo.SSR.Id) = dbo.Translations.RefId AND dbo.Translations.RefType = 'T' LEFT OUTER JOIN
                         dbo.TranslatedCategories ON dbo.SSR.CategoryId = dbo.TranslatedCategories.Id AND dbo.Translations.LanguageId = dbo.TranslatedCategories.LanguageId LEFT OUTER JOIN
                         dbo.TranslatedConstants ON dbo.TranslatedConstants.Id = dbo.SSR.Type AND dbo.TranslatedConstants.LanguageId = dbo.Translations.LanguageId
GO
/****** Object:  Table [dbo].[AspNetRoles]    Script Date: 3/11/2024 8:29:35 AM ******/
SET ANSI_NULLS ON
GO
SET QUOTED_IDENTIFIER ON
GO
CREATE TABLE [dbo].[AspNetRoles](
	[Id] [nvarchar](128) NOT NULL,
	[Name] [nvarchar](256) NOT NULL,
	[Discriminator] [nvarchar](128) NOT NULL,
	[IsActive] [bit] NOT NULL,
 CONSTRAINT [PK_dbo.AspNetRoles] PRIMARY KEY CLUSTERED 
(
	[Id] ASC
)WITH (PAD_INDEX = OFF, STATISTICS_NORECOMPUTE = OFF, IGNORE_DUP_KEY = OFF, ALLOW_ROW_LOCKS = ON, ALLOW_PAGE_LOCKS = ON, OPTIMIZE_FOR_SEQUENTIAL_KEY = OFF) ON [PRIMARY]
) ON [PRIMARY]
GO
/****** Object:  Table [dbo].[AspNetUserClaims]    Script Date: 3/11/2024 8:29:35 AM ******/
SET ANSI_NULLS ON
GO
SET QUOTED_IDENTIFIER ON
GO
CREATE TABLE [dbo].[AspNetUserClaims](
	[Id] [int] IDENTITY(1,1) NOT NULL,
	[UserId] [nvarchar](128) NOT NULL,
	[ClaimType] [nvarchar](max) NULL,
	[ClaimValue] [nvarchar](max) NULL,
 CONSTRAINT [PK_dbo.AspNetUserClaims] PRIMARY KEY CLUSTERED 
(
	[Id] ASC
)WITH (PAD_INDEX = OFF, STATISTICS_NORECOMPUTE = OFF, IGNORE_DUP_KEY = OFF, ALLOW_ROW_LOCKS = ON, ALLOW_PAGE_LOCKS = ON, OPTIMIZE_FOR_SEQUENTIAL_KEY = OFF) ON [PRIMARY]
) ON [PRIMARY] TEXTIMAGE_ON [PRIMARY]
GO
/****** Object:  Table [dbo].[AspNetUserLogins]    Script Date: 3/11/2024 8:29:35 AM ******/
SET ANSI_NULLS ON
GO
SET QUOTED_IDENTIFIER ON
GO
CREATE TABLE [dbo].[AspNetUserLogins](
	[LoginProvider] [nvarchar](128) NOT NULL,
	[ProviderKey] [nvarchar](128) NOT NULL,
	[UserId] [nvarchar](128) NOT NULL,
 CONSTRAINT [PK_dbo.AspNetUserLogins] PRIMARY KEY CLUSTERED 
(
	[LoginProvider] ASC,
	[ProviderKey] ASC,
	[UserId] ASC
)WITH (PAD_INDEX = OFF, STATISTICS_NORECOMPUTE = OFF, IGNORE_DUP_KEY = OFF, ALLOW_ROW_LOCKS = ON, ALLOW_PAGE_LOCKS = ON, OPTIMIZE_FOR_SEQUENTIAL_KEY = OFF) ON [PRIMARY]
) ON [PRIMARY]
GO
/****** Object:  Table [dbo].[AspNetUserRoles]    Script Date: 3/11/2024 8:29:35 AM ******/
SET ANSI_NULLS ON
GO
SET QUOTED_IDENTIFIER ON
GO
CREATE TABLE [dbo].[AspNetUserRoles](
	[UserId] [nvarchar](128) NOT NULL,
	[RoleId] [nvarchar](128) NOT NULL,
 CONSTRAINT [PK_dbo.AspNetUserRoles] PRIMARY KEY CLUSTERED 
(
	[UserId] ASC,
	[RoleId] ASC
)WITH (PAD_INDEX = OFF, STATISTICS_NORECOMPUTE = OFF, IGNORE_DUP_KEY = OFF, ALLOW_ROW_LOCKS = ON, ALLOW_PAGE_LOCKS = ON, OPTIMIZE_FOR_SEQUENTIAL_KEY = OFF) ON [PRIMARY]
) ON [PRIMARY]
GO
/****** Object:  Table [dbo].[AspNetUsers]    Script Date: 3/11/2024 8:29:35 AM ******/
SET ANSI_NULLS ON
GO
SET QUOTED_IDENTIFIER ON
GO
CREATE TABLE [dbo].[AspNetUsers](
	[Id] [nvarchar](128) NOT NULL,
	[Email] [nvarchar](256) NULL,
	[EmailConfirmed] [bit] NOT NULL,
	[PasswordHash] [nvarchar](max) NULL,
	[SecurityStamp] [nvarchar](max) NULL,
	[PhoneNumber] [nvarchar](max) NULL,
	[PhoneNumberConfirmed] [bit] NULL,
	[TwoFactorEnabled] [bit] NOT NULL,
	[LockoutEndDateUtc] [datetime] NULL,
	[LockoutEnabled] [bit] NOT NULL,
	[AccessFailedCount] [int] NOT NULL,
	[UserName] [nvarchar](256) NOT NULL,
	[EnglishFullName] [nvarchar](512) NULL,
	[ArabicFullName] [nvarchar](512) NULL,
	[IsActive] [bit] NOT NULL,
	[EntityId] [uniqueidentifier] NULL,
	[AssistantId] [nvarchar](128) NULL,
	[ManagerId] [nvarchar](128) NULL,
	[IsManager] [bit] NULL,
	[PreferredLanguage] [int] NULL,
	[NationalityId] [int] NULL,
	[EnglishJobPosition] [nvarchar](512) NULL,
	[ArabicJobPosition] [nvarchar](512) NULL,
	[EmployeeNumber] [nvarchar](50) NULL,
	[Signature] [xml] NULL,
	[ProfileImage] [xml] NULL,
	[Emirate] [int] NULL,
	[SyncDate] [datetime] NULL,
	[SyncStatus] [int] NULL,
	[SyncNotes] [nvarchar](2000) NULL,
	[MobileNumber] [nvarchar](500) NULL,
	[SecretQuestion] [varchar](50) NULL,
	[SecretAnswer] [nvarchar](200) NULL,
	[EmiratesID] [varchar](18) NULL,
	[Gender] [varchar](1) NULL,
	[IsApproved] [bit] NOT NULL,
	[UserType] [varchar](50) NULL,
	[EnableAuthentication] [bit] NULL,
	[UAEPassUserId] [nvarchar](128) NULL,
	[Channel] [varchar](1) NULL,
	[ProfileUpdated] [bit] NOT NULL,
	[LastLoginDate] [datetime] NULL,
	[IsDisablePerson] [bit] NULL,
	[CustomerType] [nvarchar](5) NULL,
	[MaritalStatus] [nvarchar](5) NULL,
	[DateOfBirth] [datetime] NULL,
	[EnglishFirstName] [nvarchar](512) NULL,
	[EnglishLastName] [nvarchar](512) NULL,
	[ArabicFirstName] [nvarchar](512) NULL,
	[ArabicLastName] [nvarchar](512) NULL,
	[CompanyEnglishName] [nvarchar](512) NULL,
	[CompanyArabicName] [nvarchar](512) NULL,
 CONSTRAINT [PK_dbo.AspNetUsers] PRIMARY KEY CLUSTERED 
(
	[Id] ASC
)WITH (PAD_INDEX = OFF, STATISTICS_NORECOMPUTE = OFF, IGNORE_DUP_KEY = OFF, ALLOW_ROW_LOCKS = ON, ALLOW_PAGE_LOCKS = ON, OPTIMIZE_FOR_SEQUENTIAL_KEY = OFF) ON [PRIMARY]
) ON [PRIMARY] TEXTIMAGE_ON [PRIMARY]
GO
/****** Object:  Table [dbo].[ConstantGroups]    Script Date: 3/11/2024 8:29:35 AM ******/
SET ANSI_NULLS ON
GO
SET QUOTED_IDENTIFIER ON
GO
CREATE TABLE [dbo].[ConstantGroups](
	[Id] [int] IDENTITY(1,1) NOT NULL,
	[Name] [nvarchar](200) NOT NULL,
	[IsSystem] [bit] NOT NULL,
	[DateCreated] [datetime] NOT NULL,
	[CreatedBy] [nvarchar](128) NOT NULL,
	[LastUpdated] [datetime] NOT NULL,
	[LastUpdatedBy] [nvarchar](128) NOT NULL,
 CONSTRAINT [PK_ConstantGroups] PRIMARY KEY CLUSTERED 
(
	[Id] ASC
)WITH (PAD_INDEX = OFF, STATISTICS_NORECOMPUTE = OFF, IGNORE_DUP_KEY = OFF, ALLOW_ROW_LOCKS = ON, ALLOW_PAGE_LOCKS = ON, OPTIMIZE_FOR_SEQUENTIAL_KEY = OFF) ON [PRIMARY]
) ON [PRIMARY]
GO
/****** Object:  Table [dbo].[ContentFeedbacks]    Script Date: 3/11/2024 8:29:35 AM ******/
SET ANSI_NULLS ON
GO
SET QUOTED_IDENTIFIER ON
GO
CREATE TABLE [dbo].[ContentFeedbacks](
	[Id] [uniqueidentifier] NOT NULL,
	[IsHelpfulContent] [char](1) NOT NULL,
	[Name] [nvarchar](256) NULL,
	[Email] [nvarchar](256) NULL,
	[Comment] [nvarchar](max) NULL,
	[RefId] [nvarchar](50) NOT NULL,
	[RefType] [char](1) NOT NULL,
	[IPAddress] [nvarchar](256) NOT NULL,
	[UserAgent] [nvarchar](512) NOT NULL,
	[DateCreated] [datetime] NOT NULL,
 CONSTRAINT [PK_ContentFeedBacks] PRIMARY KEY CLUSTERED 
(
	[Id] ASC
)WITH (PAD_INDEX = OFF, STATISTICS_NORECOMPUTE = OFF, IGNORE_DUP_KEY = OFF, ALLOW_ROW_LOCKS = ON, ALLOW_PAGE_LOCKS = ON, OPTIMIZE_FOR_SEQUENTIAL_KEY = OFF) ON [PRIMARY]
) ON [PRIMARY] TEXTIMAGE_ON [PRIMARY]
GO
/****** Object:  Table [dbo].[Contents]    Script Date: 3/11/2024 8:29:35 AM ******/
SET ANSI_NULLS ON
GO
SET QUOTED_IDENTIFIER ON
GO
CREATE TABLE [dbo].[Contents](
	[Id] [uniqueidentifier] NOT NULL,
	[LanguageId] [int] NOT NULL,
	[LocalizationGroupId] [uniqueidentifier] NOT NULL,
	[ContentTypeId] [int] NOT NULL,
	[Title] [nvarchar](1000) NOT NULL,
	[IntroText] [nvarchar](max) NULL,
	[FullText] [nvarchar](max) NULL,
	[Attributes] [xml] SPARSE  NULL,
	[RelatedItems] [xml] SPARSE  NULL,
	[ItemDate1] [datetime] SPARSE  NULL,
	[ItemDate2] [datetime] SPARSE  NULL,
	[IsRecurring] [bit] NOT NULL,
	[Slug] [nvarchar](128) NULL,
	[ArchiveDateFrom] [datetime] NULL,
	[ArchiveDateTo] [datetime] NULL,
	[PublishDateFrom] [datetime] NULL,
	[PublishDateTo] [datetime] NULL,
	[IsLocked] [bit] NOT NULL,
	[LockedBy] [uniqueidentifier] NULL,
	[LockDate] [datetime] NULL,
	[Images] [xml] SPARSE  NULL,
	[Likes] [int] SPARSE  NULL,
	[Dislikes] [int] SPARSE  NULL,
	[IsFullyTranslated] [bit] SPARSE  NULL,
	[CategoriesIds] [nvarchar](1024) SPARSE  NULL,
	[ContentThemeIds] [nvarchar](1024) NULL,
	[FullUrl] [nvarchar](256) SPARSE  NULL,
	[ViewsCount] [int] SPARSE  NULL,
	[MetaPageTitle] [nvarchar](1024) NULL,
	[MetaDescription] [nvarchar](2048) NULL,
	[MetaKeywords] [nvarchar](2048) NULL,
	[WorkflowStatus] [nvarchar](128) SPARSE  NULL,
	[WorkflowSummary] [xml] SPARSE  NULL,
	[DateCreated] [datetime] NOT NULL,
	[LastUpdateDate] [datetime] NOT NULL,
	[LastUpdatedBy] [nvarchar](128) NOT NULL,
	[CreatedBy] [nvarchar](128) NOT NULL,
	[IsArchived] [bit] NOT NULL,
	[IsSticky] [bit] NOT NULL,
	[IsPublished] [bit] NOT NULL,
	[IsDeleted] [bit] NOT NULL,
 CONSTRAINT [PK_ic_Contents] PRIMARY KEY CLUSTERED 
(
	[Id] ASC
)WITH (PAD_INDEX = OFF, STATISTICS_NORECOMPUTE = OFF, IGNORE_DUP_KEY = OFF, ALLOW_ROW_LOCKS = ON, ALLOW_PAGE_LOCKS = ON, OPTIMIZE_FOR_SEQUENTIAL_KEY = OFF) ON [PRIMARY]
) ON [PRIMARY] TEXTIMAGE_ON [PRIMARY]
GO
/****** Object:  Table [dbo].[EventRegistrations]    Script Date: 3/11/2024 8:29:35 AM ******/
SET ANSI_NULLS ON
GO
SET QUOTED_IDENTIFIER ON
GO
CREATE TABLE [dbo].[EventRegistrations](
	[Id] [uniqueidentifier] NOT NULL,
	[FirstName] [nvarchar](256) NOT NULL,
	[LastName] [nvarchar](256) NOT NULL,
	[EmailAddress] [nvarchar](256) NOT NULL,
	[MobileNumber] [nvarchar](50) NOT NULL,
	[EventId] [uniqueidentifier] NOT NULL,
	[Position] [nvarchar](256) NULL,
	[PlaceOfWork] [nvarchar](256) NULL,
	[EmiratesId] [nvarchar](50) NULL,
	[IsPreviousEventAttended] [bit] NOT NULL,
	[OtherChannels] [nvarchar](512) NULL,
	[Status] [char](1) NOT NULL,
	[ActionedBy] [nvarchar](50) NULL,
	[ActionedDate] [datetime] NULL,
	[RejectionReason] [nvarchar](max) NULL,
	[IsUserNotified] [bit] NOT NULL,
	[DateCreated] [datetime] NOT NULL,
	[LastUpdated] [datetime] NULL,
	[LastUpdatedBy] [nvarchar](128) NULL,
	[RefCode] [nvarchar](128) NULL,
	[EventAccessChannels] [nvarchar](max) NULL,
	[LanguageId] [int] NULL,
	[Sequence] [bigint] IDENTITY(1,1) NOT NULL,
 CONSTRAINT [PK_EventRegistration] PRIMARY KEY CLUSTERED 
(
	[Id] ASC
)WITH (PAD_INDEX = OFF, STATISTICS_NORECOMPUTE = OFF, IGNORE_DUP_KEY = OFF, ALLOW_ROW_LOCKS = ON, ALLOW_PAGE_LOCKS = ON, OPTIMIZE_FOR_SEQUENTIAL_KEY = OFF) ON [PRIMARY]
) ON [PRIMARY] TEXTIMAGE_ON [PRIMARY]
GO
/****** Object:  Table [dbo].[FormSubmissions]    Script Date: 3/11/2024 8:29:35 AM ******/
SET ANSI_NULLS ON
GO
SET QUOTED_IDENTIFIER ON
GO
CREATE TABLE [dbo].[FormSubmissions](
	[Id] [uniqueidentifier] NOT NULL,
	[FormId] [int] NOT NULL,
	[RefNumber] [varchar](50) NOT NULL,
	[UserAgent] [nvarchar](512) NOT NULL,
	[IPAddress] [nvarchar](256) NOT NULL,
	[FormValues] [xml] NOT NULL,
	[LanguageId] [int] NOT NULL,
	[WorkflowStatus] [nvarchar](128) SPARSE  NULL,
	[WorkflowSummary] [xml] SPARSE  NULL,
	[IsPublished] [bit] NOT NULL,
	[IsDeleted] [bit] NOT NULL,
	[IsRead] [bit] NOT NULL,
	[DateCreated] [datetime] NOT NULL,
	[LastUpdateDate] [datetime] NOT NULL,
	[CreatedBy] [uniqueidentifier] NULL,
	[LastUpdatedBy] [uniqueidentifier] NULL,
	[Sequence] [bigint] IDENTITY(1,1) NOT NULL,
 CONSTRAINT [pk_FormSubmission] PRIMARY KEY CLUSTERED 
(
	[Id] ASC
)WITH (PAD_INDEX = OFF, STATISTICS_NORECOMPUTE = OFF, IGNORE_DUP_KEY = OFF, ALLOW_ROW_LOCKS = ON, ALLOW_PAGE_LOCKS = ON, OPTIMIZE_FOR_SEQUENTIAL_KEY = OFF) ON [PRIMARY]
) ON [PRIMARY] TEXTIMAGE_ON [PRIMARY]
GO
/****** Object:  Table [dbo].[LibraryReservations]    Script Date: 3/11/2024 8:29:35 AM ******/
SET ANSI_NULLS ON
GO
SET QUOTED_IDENTIFIER ON
GO
CREATE TABLE [dbo].[LibraryReservations](
	[Id] [uniqueidentifier] NOT NULL,
	[FullName] [nvarchar](512) NOT NULL,
	[MobileNumber] [nvarchar](128) NOT NULL,
	[EmailAddress] [nvarchar](512) NOT NULL,
	[Notes] [nvarchar](1000) NULL,
	[DateCreated] [datetime] NOT NULL,
	[CreatedBy] [nvarchar](128) NULL,
	[LastUpdated] [datetime] NULL,
	[LastUpdatedBy] [nvarchar](128) NULL,
	[ReservationDate] [datetime] NOT NULL,
	[UserId] [nvarchar](128) NULL,
 CONSTRAINT [PK_LibraryReservation] PRIMARY KEY CLUSTERED 
(
	[Id] ASC
)WITH (PAD_INDEX = OFF, STATISTICS_NORECOMPUTE = OFF, IGNORE_DUP_KEY = OFF, ALLOW_ROW_LOCKS = ON, ALLOW_PAGE_LOCKS = ON, OPTIMIZE_FOR_SEQUENTIAL_KEY = OFF) ON [PRIMARY]
) ON [PRIMARY]
GO
/****** Object:  Table [dbo].[LibraryReservationSlots]    Script Date: 3/11/2024 8:29:35 AM ******/
SET ANSI_NULLS ON
GO
SET QUOTED_IDENTIFIER ON
GO
CREATE TABLE [dbo].[LibraryReservationSlots](
	[Id] [uniqueidentifier] NOT NULL,
	[SlotIndex] [int] NOT NULL,
	[LibraryReservationId] [uniqueidentifier] NOT NULL,
	[ReservationDate] [datetime] NOT NULL,
 CONSTRAINT [PK_LibraryReservationSlots_1] PRIMARY KEY CLUSTERED 
(
	[Id] ASC
)WITH (PAD_INDEX = OFF, STATISTICS_NORECOMPUTE = OFF, IGNORE_DUP_KEY = OFF, ALLOW_ROW_LOCKS = ON, ALLOW_PAGE_LOCKS = ON, OPTIMIZE_FOR_SEQUENTIAL_KEY = OFF) ON [PRIMARY]
) ON [PRIMARY]
GO
/****** Object:  Table [dbo].[Logs]    Script Date: 3/11/2024 8:29:35 AM ******/
SET ANSI_NULLS ON
GO
SET QUOTED_IDENTIFIER ON
GO
CREATE TABLE [dbo].[Logs](
	[Id] [uniqueidentifier] NOT NULL,
	[Title] [nvarchar](2000) NULL,
	[ControllerName] [nvarchar](128) NULL,
	[RefTable] [char](1) NOT NULL,
	[RefId] [varchar](50) NULL,
	[RefHistoryId] [varchar](50) NULL,
	[ActionType] [varchar](50) NOT NULL,
	[ActivityDate] [datetime] NOT NULL,
	[UserId] [nvarchar](128) NULL,
	[Comments] [nvarchar](1000) NULL,
	[IPAddress] [varchar](50) NULL,
	[UserAgent] [nvarchar](250) NULL,
 CONSTRAINT [PK_Logs] PRIMARY KEY CLUSTERED 
(
	[Id] ASC
)WITH (PAD_INDEX = OFF, STATISTICS_NORECOMPUTE = OFF, IGNORE_DUP_KEY = OFF, ALLOW_ROW_LOCKS = ON, ALLOW_PAGE_LOCKS = ON, OPTIMIZE_FOR_SEQUENTIAL_KEY = OFF) ON [PRIMARY]
) ON [PRIMARY]
GO
/****** Object:  Table [dbo].[NewsLetterSubscription]    Script Date: 3/11/2024 8:29:35 AM ******/
SET ANSI_NULLS ON
GO
SET QUOTED_IDENTIFIER ON
GO
CREATE TABLE [dbo].[NewsLetterSubscription](
	[Id] [uniqueidentifier] NOT NULL,
	[Email] [nvarchar](255) NOT NULL,
	[IsActive] [bit] NOT NULL,
	[DateCreated] [datetime] NOT NULL,
	[LanguageId] [int] NOT NULL,
	[CreatedBy] [nvarchar](128) NULL,
	[LastUpdated] [datetime] NULL,
	[LastUpdatedBy] [nvarchar](128) NULL,
 CONSTRAINT [PK_NewsletterSubscription] PRIMARY KEY CLUSTERED 
(
	[Id] ASC
)WITH (PAD_INDEX = OFF, STATISTICS_NORECOMPUTE = OFF, IGNORE_DUP_KEY = OFF, ALLOW_ROW_LOCKS = ON, ALLOW_PAGE_LOCKS = ON, OPTIMIZE_FOR_SEQUENTIAL_KEY = OFF) ON [PRIMARY]
) ON [PRIMARY]
GO
/****** Object:  Table [dbo].[NotificationDefinitions]    Script Date: 3/11/2024 8:29:35 AM ******/
SET ANSI_NULLS ON
GO
SET QUOTED_IDENTIFIER ON
GO
CREATE TABLE [dbo].[NotificationDefinitions](
	[Id] [uniqueidentifier] NOT NULL,
	[Title] [nvarchar](256) NOT NULL,
	[UserId] [nvarchar](128) NULL,
	[ApplicationId] [uniqueidentifier] NULL,
	[ActionUrl] [nvarchar](1024) NULL,
	[DateCreated] [datetime] NOT NULL,
	[ScheduledStartDate] [datetime] NOT NULL,
	[ScheduledNextDate] [datetime] NOT NULL,
	[IsRepeated] [bit] NOT NULL,
	[FrequencyIntervalRate] [char](1) NOT NULL,
	[FrequencyInterval] [int] NULL,
	[Status] [char](1) NOT NULL,
	[VerifyFunction] [varchar](512) NULL,
	[NotificationType] [char](1) NOT NULL,
	[TitleResource] [nvarchar](512) NULL,
	[BodyResource] [nvarchar](512) NULL,
	[EmailBodyResource] [nvarchar](512) NULL,
	[ArabicTitle] [nvarchar](512) NULL,
	[ArabicBody] [nvarchar](max) NULL,
	[ArabicEmailBody] [nvarchar](max) NULL,
	[EnglishTitle] [nvarchar](512) NULL,
	[EnglishBody] [nvarchar](max) NULL,
	[EnglishEmailBody] [nvarchar](max) NULL,
 CONSTRAINT [PK_NotificationDefinitions] PRIMARY KEY CLUSTERED 
(
	[Id] ASC
)WITH (PAD_INDEX = OFF, STATISTICS_NORECOMPUTE = OFF, IGNORE_DUP_KEY = OFF, ALLOW_ROW_LOCKS = ON, ALLOW_PAGE_LOCKS = ON, OPTIMIZE_FOR_SEQUENTIAL_KEY = OFF) ON [PRIMARY]
) ON [PRIMARY] TEXTIMAGE_ON [PRIMARY]
GO
/****** Object:  Table [dbo].[NotificationTransactions]    Script Date: 3/11/2024 8:29:35 AM ******/
SET ANSI_NULLS ON
GO
SET QUOTED_IDENTIFIER ON
GO
CREATE TABLE [dbo].[NotificationTransactions](
	[Id] [uniqueidentifier] NOT NULL,
	[NotificationDefinitionId] [uniqueidentifier] NOT NULL,
	[Status] [char](1) NOT NULL,
	[DateCreated] [datetime] NOT NULL,
	[Notes] [nvarchar](max) NULL,
 CONSTRAINT [PK_NotificationTransactions] PRIMARY KEY CLUSTERED 
(
	[Id] ASC
)WITH (PAD_INDEX = OFF, STATISTICS_NORECOMPUTE = OFF, IGNORE_DUP_KEY = OFF, ALLOW_ROW_LOCKS = ON, ALLOW_PAGE_LOCKS = ON, OPTIMIZE_FOR_SEQUENTIAL_KEY = OFF) ON [PRIMARY]
) ON [PRIMARY] TEXTIMAGE_ON [PRIMARY]
GO
/****** Object:  Table [dbo].[ProductOrderItems]    Script Date: 3/11/2024 8:29:35 AM ******/
SET ANSI_NULLS ON
GO
SET QUOTED_IDENTIFIER ON
GO
CREATE TABLE [dbo].[ProductOrderItems](
	[Id] [uniqueidentifier] NOT NULL,
	[ProductOrderId] [uniqueidentifier] NOT NULL,
	[ProductId] [uniqueidentifier] NOT NULL,
	[ProductOptionId] [uniqueidentifier] NOT NULL,
	[Quantity] [int] NOT NULL,
	[LastUpdatedDate] [datetime] NOT NULL,
	[DateCreated] [datetime] NOT NULL,
	[DownloadLimit] [int] NULL,
	[DownloadExpiryDays] [int] NULL,
	[ItemPrice] [float] NOT NULL,
 CONSTRAINT [PK_ECommerceProductOrderItems] PRIMARY KEY CLUSTERED 
(
	[Id] ASC
)WITH (PAD_INDEX = OFF, STATISTICS_NORECOMPUTE = OFF, IGNORE_DUP_KEY = OFF, ALLOW_ROW_LOCKS = ON, ALLOW_PAGE_LOCKS = ON, OPTIMIZE_FOR_SEQUENTIAL_KEY = OFF) ON [PRIMARY]
) ON [PRIMARY]
GO
/****** Object:  Table [dbo].[ProductOrders]    Script Date: 3/11/2024 8:29:35 AM ******/
SET ANSI_NULLS ON
GO
SET QUOTED_IDENTIFIER ON
GO
CREATE TABLE [dbo].[ProductOrders](
	[Id] [uniqueidentifier] NOT NULL,
	[Status] [char](1) NOT NULL,
	[RefCode] [nvarchar](128) NULL,
	[LastUpdatedDate] [datetime] NULL,
	[DateCreated] [datetime] NOT NULL,
	[Vat] [float] NULL,
	[TotalPrice] [float] NULL,
	[ShippingPrice] [float] NULL,
	[UserId] [nvarchar](128) NULL,
	[UserShippingAddressId] [uniqueidentifier] NULL,
	[PaymentStatus] [char](1) NOT NULL,
	[PaymentLasActionDate] [datetime] NULL,
	[ShippingTrakingNumber] [nvarchar](256) NULL,
	[LastUpdatedBy] [nvarchar](128) NULL,
	[Sequence] [bigint] IDENTITY(1,1) NOT NULL,
	[IsCopied] [bit] NOT NULL,
 CONSTRAINT [PK_ECommerceProductOrders] PRIMARY KEY CLUSTERED 
(
	[Id] ASC
)WITH (PAD_INDEX = OFF, STATISTICS_NORECOMPUTE = OFF, IGNORE_DUP_KEY = OFF, ALLOW_ROW_LOCKS = ON, ALLOW_PAGE_LOCKS = ON, OPTIMIZE_FOR_SEQUENTIAL_KEY = OFF) ON [PRIMARY]
) ON [PRIMARY]
GO
/****** Object:  Table [dbo].[ProductOrderTransactions]    Script Date: 3/11/2024 8:29:35 AM ******/
SET ANSI_NULLS ON
GO
SET QUOTED_IDENTIFIER ON
GO
CREATE TABLE [dbo].[ProductOrderTransactions](
	[Id] [uniqueidentifier] NOT NULL,
	[ProductOrderId] [uniqueidentifier] NOT NULL,
	[TransactionId] [nvarchar](256) NOT NULL,
	[ResponseCode] [int] NOT NULL,
	[ResponseDescription] [nvarchar](max) NULL,
	[ApprovalCode] [nvarchar](256) NULL,
	[Account] [nvarchar](512) NULL,
	[UniqueID] [nvarchar](512) NULL,
	[Amount] [float] NULL,
	[Currency] [nvarchar](50) NULL,
	[Data] [xml] NULL,
	[DateCreated] [datetime] NOT NULL,
	[LastUpdateDate] [datetime] NULL,
	[Status] [char](1) NOT NULL,
 CONSTRAINT [PK_ProductOrderTransactions] PRIMARY KEY CLUSTERED 
(
	[Id] ASC
)WITH (PAD_INDEX = OFF, STATISTICS_NORECOMPUTE = OFF, IGNORE_DUP_KEY = OFF, ALLOW_ROW_LOCKS = ON, ALLOW_PAGE_LOCKS = ON, OPTIMIZE_FOR_SEQUENTIAL_KEY = OFF) ON [PRIMARY]
) ON [PRIMARY] TEXTIMAGE_ON [PRIMARY]
GO
/****** Object:  Table [dbo].[ProductStockNotifications]    Script Date: 3/11/2024 8:29:35 AM ******/
SET ANSI_NULLS ON
GO
SET QUOTED_IDENTIFIER ON
GO
CREATE TABLE [dbo].[ProductStockNotifications](
	[Id] [uniqueidentifier] NOT NULL,
	[UserId] [nvarchar](50) NOT NULL,
	[IsNotified] [bit] NOT NULL,
	[NotifiedDate] [datetime] NULL,
	[DateCreated] [datetime] NOT NULL,
	[LastUpdated] [datetime] NOT NULL,
	[ProductId] [uniqueidentifier] NOT NULL,
	[ProductOptionId] [uniqueidentifier] NOT NULL,
 CONSTRAINT [PK_StockNotifications] PRIMARY KEY CLUSTERED 
(
	[Id] ASC
)WITH (PAD_INDEX = OFF, STATISTICS_NORECOMPUTE = OFF, IGNORE_DUP_KEY = OFF, ALLOW_ROW_LOCKS = ON, ALLOW_PAGE_LOCKS = ON, OPTIMIZE_FOR_SEQUENTIAL_KEY = OFF) ON [PRIMARY]
) ON [PRIMARY]
GO
/****** Object:  Table [dbo].[QueuedEmails]    Script Date: 3/11/2024 8:29:35 AM ******/
SET ANSI_NULLS ON
GO
SET QUOTED_IDENTIFIER ON
GO
CREATE TABLE [dbo].[QueuedEmails](
	[Id] [int] IDENTITY(1,1) NOT NULL,
	[ActionId] [int] NULL,
	[UserEmail] [nvarchar](200) NULL,
	[UserId] [nvarchar](128) NULL,
	[Subject] [nvarchar](500) NOT NULL,
	[Body] [nvarchar](max) NOT NULL,
	[SentDate] [datetime] NULL,
	[SentTries] [int] NOT NULL,
	[DateCreated] [datetime] NOT NULL,
	[CreatedBy] [nvarchar](128) NULL,
	[IsDeleted] [bit] NOT NULL,
 CONSTRAINT [PK_QueuedEmails] PRIMARY KEY CLUSTERED 
(
	[Id] ASC
)WITH (PAD_INDEX = OFF, STATISTICS_NORECOMPUTE = OFF, IGNORE_DUP_KEY = OFF, ALLOW_ROW_LOCKS = ON, ALLOW_PAGE_LOCKS = ON, OPTIMIZE_FOR_SEQUENTIAL_KEY = OFF) ON [PRIMARY]
) ON [PRIMARY] TEXTIMAGE_ON [PRIMARY]
GO
/****** Object:  Table [dbo].[QueuedNewsLetters]    Script Date: 3/11/2024 8:29:35 AM ******/
SET ANSI_NULLS ON
GO
SET QUOTED_IDENTIFIER ON
GO
CREATE TABLE [dbo].[QueuedNewsLetters](
	[Id] [int] IDENTITY(1,1) NOT NULL,
	[RefType] [char](1) NOT NULL,
	[RefId] [nvarchar](50) NOT NULL,
	[DateCreated] [datetime] NOT NULL,
 CONSTRAINT [PK_QueuedNewsLetters] PRIMARY KEY CLUSTERED 
(
	[Id] ASC
)WITH (PAD_INDEX = OFF, STATISTICS_NORECOMPUTE = OFF, IGNORE_DUP_KEY = OFF, ALLOW_ROW_LOCKS = ON, ALLOW_PAGE_LOCKS = ON, OPTIMIZE_FOR_SEQUENTIAL_KEY = OFF) ON [PRIMARY]
) ON [PRIMARY]
GO
/****** Object:  Table [dbo].[Ratings]    Script Date: 3/11/2024 8:29:35 AM ******/
SET ANSI_NULLS ON
GO
SET QUOTED_IDENTIFIER ON
GO
CREATE TABLE [dbo].[Ratings](
	[Id] [uniqueidentifier] NOT NULL,
	[RefId] [nvarchar](50) NOT NULL,
	[RefType] [char](1) NOT NULL,
	[RatingValue] [float] NOT NULL,
	[DateCreated] [datetime] NOT NULL,
	[IPAddress] [nvarchar](256) NOT NULL,
	[UserAgent] [nvarchar](512) NOT NULL,
 CONSTRAINT [PK_Ratings] PRIMARY KEY CLUSTERED 
(
	[Id] ASC
)WITH (PAD_INDEX = OFF, STATISTICS_NORECOMPUTE = OFF, IGNORE_DUP_KEY = OFF, ALLOW_ROW_LOCKS = ON, ALLOW_PAGE_LOCKS = ON, OPTIMIZE_FOR_SEQUENTIAL_KEY = OFF) ON [PRIMARY]
) ON [PRIMARY]
GO
/****** Object:  Table [dbo].[Settings]    Script Date: 3/11/2024 8:29:35 AM ******/
SET ANSI_NULLS ON
GO
SET QUOTED_IDENTIFIER ON
GO
CREATE TABLE [dbo].[Settings](
	[Id] [int] IDENTITY(1,1) NOT NULL,
	[SettingKey] [nvarchar](128) NOT NULL,
	[SettingValue] [nvarchar](1024) NULL,
	[DateCreated] [datetime] NOT NULL,
	[CreatedBy] [nvarchar](128) NOT NULL,
	[LastUpdated] [datetime] NOT NULL,
	[LastUpdatedBy] [nvarchar](128) NOT NULL,
 CONSTRAINT [PK_Settings] PRIMARY KEY CLUSTERED 
(
	[Id] ASC
)WITH (PAD_INDEX = OFF, STATISTICS_NORECOMPUTE = OFF, IGNORE_DUP_KEY = OFF, ALLOW_ROW_LOCKS = ON, ALLOW_PAGE_LOCKS = ON, OPTIMIZE_FOR_SEQUENTIAL_KEY = OFF) ON [PRIMARY]
) ON [PRIMARY]
GO
/****** Object:  Table [dbo].[SettingsHistory]    Script Date: 3/11/2024 8:29:35 AM ******/
SET ANSI_NULLS ON
GO
SET QUOTED_IDENTIFIER ON
GO
CREATE TABLE [dbo].[SettingsHistory](
	[Id] [int] IDENTITY(1,1) NOT NULL,
	[SettingId] [int] NOT NULL,
	[SettingKey] [nvarchar](128) NOT NULL,
	[SettingValue] [nvarchar](1024) NULL,
	[DateCreated] [datetime] NOT NULL,
	[CreatedBy] [nvarchar](128) NOT NULL,
	[LastUpdated] [datetime] NOT NULL,
	[LastUpdatedBy] [nvarchar](128) NOT NULL,
	[ActivityDate] [datetime] NOT NULL,
 CONSTRAINT [PK_SettingsHistory] PRIMARY KEY CLUSTERED 
(
	[Id] ASC
)WITH (PAD_INDEX = OFF, STATISTICS_NORECOMPUTE = OFF, IGNORE_DUP_KEY = OFF, ALLOW_ROW_LOCKS = ON, ALLOW_PAGE_LOCKS = ON, OPTIMIZE_FOR_SEQUENTIAL_KEY = OFF) ON [PRIMARY]
) ON [PRIMARY]
GO
/****** Object:  Table [dbo].[UserFavorites]    Script Date: 3/11/2024 8:29:35 AM ******/
SET ANSI_NULLS ON
GO
SET QUOTED_IDENTIFIER ON
GO
CREATE TABLE [dbo].[UserFavorites](
	[Id] [uniqueidentifier] NOT NULL,
	[UserId] [nvarchar](128) NOT NULL,
	[RefId] [varchar](50) NOT NULL,
	[RefType] [char](1) NOT NULL,
	[DateCreated] [datetime] NOT NULL,
	[Url] [nvarchar](512) NULL,
 CONSTRAINT [PK_UserFavorites] PRIMARY KEY CLUSTERED 
(
	[Id] ASC
)WITH (PAD_INDEX = OFF, STATISTICS_NORECOMPUTE = OFF, IGNORE_DUP_KEY = OFF, ALLOW_ROW_LOCKS = ON, ALLOW_PAGE_LOCKS = ON, OPTIMIZE_FOR_SEQUENTIAL_KEY = OFF) ON [PRIMARY]
) ON [PRIMARY]
GO
/****** Object:  Table [dbo].[UserProducts]    Script Date: 3/11/2024 8:29:35 AM ******/
SET ANSI_NULLS ON
GO
SET QUOTED_IDENTIFIER ON
GO
CREATE TABLE [dbo].[UserProducts](
	[Id] [uniqueidentifier] NOT NULL,
	[ProductId] [uniqueidentifier] NOT NULL,
	[ProductOptionId] [uniqueidentifier] NOT NULL,
	[ProductOrderId] [uniqueidentifier] NOT NULL,
	[DownloadCount] [int] NOT NULL,
	[DateCreated] [datetime] NOT NULL,
	[DownloadExpiryDate] [datetime] NULL,
	[UserId] [nvarchar](128) NOT NULL,
	[DownloadLimit] [int] NULL,
	[DeliveryMethod] [char](1) NULL,
	[IsDelivered] [bit] NOT NULL,
 CONSTRAINT [PK_UserECommerceProducts] PRIMARY KEY CLUSTERED 
(
	[Id] ASC
)WITH (PAD_INDEX = OFF, STATISTICS_NORECOMPUTE = OFF, IGNORE_DUP_KEY = OFF, ALLOW_ROW_LOCKS = ON, ALLOW_PAGE_LOCKS = ON, OPTIMIZE_FOR_SEQUENTIAL_KEY = OFF) ON [PRIMARY]
) ON [PRIMARY]
GO
/****** Object:  Table [dbo].[UserSettings]    Script Date: 3/11/2024 8:29:35 AM ******/
SET ANSI_NULLS ON
GO
SET QUOTED_IDENTIFIER ON
GO
CREATE TABLE [dbo].[UserSettings](
	[Id] [int] IDENTITY(1,1) NOT NULL,
	[UserId] [varchar](128) NOT NULL,
	[Settings] [xml] NULL,
 CONSTRAINT [PK_UserSettings] PRIMARY KEY CLUSTERED 
(
	[Id] ASC
)WITH (PAD_INDEX = OFF, STATISTICS_NORECOMPUTE = OFF, IGNORE_DUP_KEY = OFF, ALLOW_ROW_LOCKS = ON, ALLOW_PAGE_LOCKS = ON, OPTIMIZE_FOR_SEQUENTIAL_KEY = OFF) ON [PRIMARY]
) ON [PRIMARY] TEXTIMAGE_ON [PRIMARY]
GO
/****** Object:  Table [dbo].[UserShippingAddresses]    Script Date: 3/11/2024 8:29:35 AM ******/
SET ANSI_NULLS ON
GO
SET QUOTED_IDENTIFIER ON
GO
CREATE TABLE [dbo].[UserShippingAddresses](
	[Id] [uniqueidentifier] NOT NULL,
	[FullName] [nvarchar](512) NULL,
	[MobileNumber] [nvarchar](128) NULL,
	[PhoneNumber] [nvarchar](128) NULL,
	[Email] [nvarchar](256) NULL,
	[Country] [nvarchar](512) NULL,
	[City] [nvarchar](512) NULL,
	[State] [nvarchar](512) NULL,
	[Address] [nvarchar](1024) NULL,
	[UnitNumber] [nvarchar](512) NULL,
	[Notes] [nvarchar](max) NULL,
	[Latitude] [nvarchar](512) NULL,
	[Longitude] [nvarchar](512) NULL,
	[UserId] [nvarchar](128) NULL,
	[IsLastUsedVersion] [bit] NOT NULL,
	[CreatedBy] [nvarchar](128) NULL,
	[DateCreated] [datetime] NOT NULL,
	[LastUpdated] [datetime] NULL,
	[LastUpdatedBy] [nvarchar](128) NULL,
	[CountryData] [xml] NULL,
	[CityData] [xml] NULL,
 CONSTRAINT [PK_UserShippingAddresses] PRIMARY KEY CLUSTERED 
(
	[Id] ASC
)WITH (PAD_INDEX = OFF, STATISTICS_NORECOMPUTE = OFF, IGNORE_DUP_KEY = OFF, ALLOW_ROW_LOCKS = ON, ALLOW_PAGE_LOCKS = ON, OPTIMIZE_FOR_SEQUENTIAL_KEY = OFF) ON [PRIMARY]
) ON [PRIMARY] TEXTIMAGE_ON [PRIMARY]
GO
/****** Object:  Table [dbo].[VisitorCounter]    Script Date: 3/11/2024 8:29:35 AM ******/
SET ANSI_NULLS ON
GO
SET QUOTED_IDENTIFIER ON
GO
CREATE TABLE [dbo].[VisitorCounter](
	[Id] [int] NOT NULL,
	[Count] [int] NULL,
	[IsActive] [bit] NOT NULL,
 CONSTRAINT [PK_VisitorCounter] PRIMARY KEY CLUSTERED 
(
	[Id] ASC
)WITH (PAD_INDEX = OFF, STATISTICS_NORECOMPUTE = OFF, IGNORE_DUP_KEY = OFF, ALLOW_ROW_LOCKS = ON, ALLOW_PAGE_LOCKS = ON, OPTIMIZE_FOR_SEQUENTIAL_KEY = OFF) ON [PRIMARY]
) ON [PRIMARY]
GO
/****** Object:  Table [dbo].[Worklists]    Script Date: 3/11/2024 8:29:35 AM ******/
SET ANSI_NULLS ON
GO
SET QUOTED_IDENTIFIER ON
GO
CREATE TABLE [dbo].[Worklists](
	[Id] [uniqueidentifier] NOT NULL,
	[WorklistType] [char](1) NOT NULL,
	[UserId] [nvarchar](128) NOT NULL,
	[ActorId] [int] NULL,
	[ApplicationId] [uniqueidentifier] NULL,
	[ReferenceNo] [varchar](30) NULL,
	[EnglishSubject] [nvarchar](200) NULL,
	[ArabicSubject] [nvarchar](200) NULL,
	[TemplateId] [int] NOT NULL,
	[DateCreated] [datetime] NOT NULL,
	[DateOpened] [datetime] NULL,
	[DateActioned] [datetime] NULL,
	[IsFlagged] [bit] NOT NULL,
	[IsDeleted] [bit] NOT NULL,
	[IsArchived] [bit] NOT NULL,
	[ActionLink] [nvarchar](500) NULL,
	[UrlToken] [nvarchar](50) NULL,
	[MessageResource] [nvarchar](128) NULL,
	[RefType] [char](1) NULL,
	[RefId] [nvarchar](50) NULL,
	[EnglishMessage] [nvarchar](max) NULL,
	[ArabicMessage] [nvarchar](max) NULL,
 CONSTRAINT [PK_Worklist] PRIMARY KEY CLUSTERED 
(
	[Id] ASC
)WITH (PAD_INDEX = OFF, STATISTICS_NORECOMPUTE = OFF, IGNORE_DUP_KEY = OFF, ALLOW_ROW_LOCKS = ON, ALLOW_PAGE_LOCKS = ON, OPTIMIZE_FOR_SEQUENTIAL_KEY = OFF) ON [PRIMARY]
) ON [PRIMARY] TEXTIMAGE_ON [PRIMARY]
GO
/****** Object:  Table [HangFire].[AggregatedCounter]    Script Date: 3/11/2024 8:29:35 AM ******/
SET ANSI_NULLS ON
GO
SET QUOTED_IDENTIFIER ON
GO
CREATE TABLE [HangFire].[AggregatedCounter](
	[Key] [nvarchar](100) NOT NULL,
	[Value] [bigint] NOT NULL,
	[ExpireAt] [datetime] NULL,
 CONSTRAINT [PK_HangFire_CounterAggregated] PRIMARY KEY CLUSTERED 
(
	[Key] ASC
)WITH (PAD_INDEX = OFF, STATISTICS_NORECOMPUTE = OFF, IGNORE_DUP_KEY = OFF, ALLOW_ROW_LOCKS = ON, ALLOW_PAGE_LOCKS = ON, OPTIMIZE_FOR_SEQUENTIAL_KEY = OFF) ON [PRIMARY]
) ON [PRIMARY]
GO
/****** Object:  Table [HangFire].[Counter]    Script Date: 3/11/2024 8:29:35 AM ******/
SET ANSI_NULLS ON
GO
SET QUOTED_IDENTIFIER ON
GO
CREATE TABLE [HangFire].[Counter](
	[Key] [nvarchar](100) NOT NULL,
	[Value] [int] NOT NULL,
	[ExpireAt] [datetime] NULL
) ON [PRIMARY]
GO
/****** Object:  Table [HangFire].[Hash]    Script Date: 3/11/2024 8:29:35 AM ******/
SET ANSI_NULLS ON
GO
SET QUOTED_IDENTIFIER ON
GO
CREATE TABLE [HangFire].[Hash](
	[Key] [nvarchar](100) NOT NULL,
	[Field] [nvarchar](100) NOT NULL,
	[Value] [nvarchar](max) NULL,
	[ExpireAt] [datetime2](7) NULL,
 CONSTRAINT [PK_HangFire_Hash] PRIMARY KEY CLUSTERED 
(
	[Key] ASC,
	[Field] ASC
)WITH (PAD_INDEX = OFF, STATISTICS_NORECOMPUTE = OFF, IGNORE_DUP_KEY = OFF, ALLOW_ROW_LOCKS = ON, ALLOW_PAGE_LOCKS = ON, OPTIMIZE_FOR_SEQUENTIAL_KEY = OFF) ON [PRIMARY]
) ON [PRIMARY] TEXTIMAGE_ON [PRIMARY]
GO
/****** Object:  Table [HangFire].[Job]    Script Date: 3/11/2024 8:29:35 AM ******/
SET ANSI_NULLS ON
GO
SET QUOTED_IDENTIFIER ON
GO
CREATE TABLE [HangFire].[Job](
	[Id] [bigint] IDENTITY(1,1) NOT NULL,
	[StateId] [bigint] NULL,
	[StateName] [nvarchar](20) NULL,
	[InvocationData] [nvarchar](max) NOT NULL,
	[Arguments] [nvarchar](max) NOT NULL,
	[CreatedAt] [datetime] NOT NULL,
	[ExpireAt] [datetime] NULL,
 CONSTRAINT [PK_HangFire_Job] PRIMARY KEY CLUSTERED 
(
	[Id] ASC
)WITH (PAD_INDEX = OFF, STATISTICS_NORECOMPUTE = OFF, IGNORE_DUP_KEY = OFF, ALLOW_ROW_LOCKS = ON, ALLOW_PAGE_LOCKS = ON, OPTIMIZE_FOR_SEQUENTIAL_KEY = OFF) ON [PRIMARY]
) ON [PRIMARY] TEXTIMAGE_ON [PRIMARY]
GO
/****** Object:  Table [HangFire].[JobParameter]    Script Date: 3/11/2024 8:29:35 AM ******/
SET ANSI_NULLS ON
GO
SET QUOTED_IDENTIFIER ON
GO
CREATE TABLE [HangFire].[JobParameter](
	[JobId] [bigint] NOT NULL,
	[Name] [nvarchar](40) NOT NULL,
	[Value] [nvarchar](max) NULL,
 CONSTRAINT [PK_HangFire_JobParameter] PRIMARY KEY CLUSTERED 
(
	[JobId] ASC,
	[Name] ASC
)WITH (PAD_INDEX = OFF, STATISTICS_NORECOMPUTE = OFF, IGNORE_DUP_KEY = OFF, ALLOW_ROW_LOCKS = ON, ALLOW_PAGE_LOCKS = ON, OPTIMIZE_FOR_SEQUENTIAL_KEY = OFF) ON [PRIMARY]
) ON [PRIMARY] TEXTIMAGE_ON [PRIMARY]
GO
/****** Object:  Table [HangFire].[JobQueue]    Script Date: 3/11/2024 8:29:35 AM ******/
SET ANSI_NULLS ON
GO
SET QUOTED_IDENTIFIER ON
GO
CREATE TABLE [HangFire].[JobQueue](
	[Id] [bigint] IDENTITY(1,1) NOT NULL,
	[JobId] [bigint] NOT NULL,
	[Queue] [nvarchar](50) NOT NULL,
	[FetchedAt] [datetime] NULL,
 CONSTRAINT [PK_HangFire_JobQueue] PRIMARY KEY CLUSTERED 
(
	[Queue] ASC,
	[Id] ASC
)WITH (PAD_INDEX = OFF, STATISTICS_NORECOMPUTE = OFF, IGNORE_DUP_KEY = OFF, ALLOW_ROW_LOCKS = ON, ALLOW_PAGE_LOCKS = ON, OPTIMIZE_FOR_SEQUENTIAL_KEY = OFF) ON [PRIMARY]
) ON [PRIMARY]
GO
/****** Object:  Table [HangFire].[List]    Script Date: 3/11/2024 8:29:35 AM ******/
SET ANSI_NULLS ON
GO
SET QUOTED_IDENTIFIER ON
GO
CREATE TABLE [HangFire].[List](
	[Id] [bigint] IDENTITY(1,1) NOT NULL,
	[Key] [nvarchar](100) NOT NULL,
	[Value] [nvarchar](max) NULL,
	[ExpireAt] [datetime] NULL,
 CONSTRAINT [PK_HangFire_List] PRIMARY KEY CLUSTERED 
(
	[Key] ASC,
	[Id] ASC
)WITH (PAD_INDEX = OFF, STATISTICS_NORECOMPUTE = OFF, IGNORE_DUP_KEY = OFF, ALLOW_ROW_LOCKS = ON, ALLOW_PAGE_LOCKS = ON, OPTIMIZE_FOR_SEQUENTIAL_KEY = OFF) ON [PRIMARY]
) ON [PRIMARY] TEXTIMAGE_ON [PRIMARY]
GO
/****** Object:  Table [HangFire].[Schema]    Script Date: 3/11/2024 8:29:35 AM ******/
SET ANSI_NULLS ON
GO
SET QUOTED_IDENTIFIER ON
GO
CREATE TABLE [HangFire].[Schema](
	[Version] [int] NOT NULL,
 CONSTRAINT [PK_HangFire_Schema] PRIMARY KEY CLUSTERED 
(
	[Version] ASC
)WITH (PAD_INDEX = OFF, STATISTICS_NORECOMPUTE = OFF, IGNORE_DUP_KEY = OFF, ALLOW_ROW_LOCKS = ON, ALLOW_PAGE_LOCKS = ON, OPTIMIZE_FOR_SEQUENTIAL_KEY = OFF) ON [PRIMARY]
) ON [PRIMARY]
GO
/****** Object:  Table [HangFire].[Server]    Script Date: 3/11/2024 8:29:35 AM ******/
SET ANSI_NULLS ON
GO
SET QUOTED_IDENTIFIER ON
GO
CREATE TABLE [HangFire].[Server](
	[Id] [nvarchar](200) NOT NULL,
	[Data] [nvarchar](max) NULL,
	[LastHeartbeat] [datetime] NOT NULL,
 CONSTRAINT [PK_HangFire_Server] PRIMARY KEY CLUSTERED 
(
	[Id] ASC
)WITH (PAD_INDEX = OFF, STATISTICS_NORECOMPUTE = OFF, IGNORE_DUP_KEY = OFF, ALLOW_ROW_LOCKS = ON, ALLOW_PAGE_LOCKS = ON, OPTIMIZE_FOR_SEQUENTIAL_KEY = OFF) ON [PRIMARY]
) ON [PRIMARY] TEXTIMAGE_ON [PRIMARY]
GO
/****** Object:  Table [HangFire].[Set]    Script Date: 3/11/2024 8:29:35 AM ******/
SET ANSI_NULLS ON
GO
SET QUOTED_IDENTIFIER ON
GO
CREATE TABLE [HangFire].[Set](
	[Key] [nvarchar](100) NOT NULL,
	[Score] [float] NOT NULL,
	[Value] [nvarchar](256) NOT NULL,
	[ExpireAt] [datetime] NULL,
 CONSTRAINT [PK_HangFire_Set] PRIMARY KEY CLUSTERED 
(
	[Key] ASC,
	[Value] ASC
)WITH (PAD_INDEX = OFF, STATISTICS_NORECOMPUTE = OFF, IGNORE_DUP_KEY = OFF, ALLOW_ROW_LOCKS = ON, ALLOW_PAGE_LOCKS = ON, OPTIMIZE_FOR_SEQUENTIAL_KEY = OFF) ON [PRIMARY]
) ON [PRIMARY]
GO
/****** Object:  Table [HangFire].[State]    Script Date: 3/11/2024 8:29:35 AM ******/
SET ANSI_NULLS ON
GO
SET QUOTED_IDENTIFIER ON
GO
CREATE TABLE [HangFire].[State](
	[Id] [bigint] IDENTITY(1,1) NOT NULL,
	[JobId] [bigint] NOT NULL,
	[Name] [nvarchar](20) NOT NULL,
	[Reason] [nvarchar](100) NULL,
	[CreatedAt] [datetime] NOT NULL,
	[Data] [nvarchar](max) NULL,
 CONSTRAINT [PK_HangFire_State] PRIMARY KEY CLUSTERED 
(
	[JobId] ASC,
	[Id] ASC
)WITH (PAD_INDEX = OFF, STATISTICS_NORECOMPUTE = OFF, IGNORE_DUP_KEY = OFF, ALLOW_ROW_LOCKS = ON, ALLOW_PAGE_LOCKS = ON, OPTIMIZE_FOR_SEQUENTIAL_KEY = OFF) ON [PRIMARY]
) ON [PRIMARY] TEXTIMAGE_ON [PRIMARY]
GO
ALTER TABLE [dbo].[AspNetUsers] ADD  CONSTRAINT [DF__AspNetUse__Profi__0CE5D100]  DEFAULT ((0)) FOR [ProfileUpdated]
GO
ALTER TABLE [dbo].[ConstantGroups] ADD  CONSTRAINT [DF_ConstantGroups_IsSystem]  DEFAULT ((0)) FOR [IsSystem]
GO
ALTER TABLE [dbo].[Constants] ADD  CONSTRAINT [DF_Constants_GroupId]  DEFAULT ((0)) FOR [GroupId]
GO
ALTER TABLE [dbo].[Contents] ADD  CONSTRAINT [DF_ic_Contents_IsPublished]  DEFAULT ((1)) FOR [IsPublished]
GO
ALTER TABLE [dbo].[ContentTypes] ADD  CONSTRAINT [DF_ContentTypes_IsPublished]  DEFAULT ((1)) FOR [IsPublished]
GO
ALTER TABLE [dbo].[Events] ADD  DEFAULT ((0)) FOR [ApprovalRequired]
GO
ALTER TABLE [dbo].[Forms] ADD  DEFAULT ((0)) FOR [GenerateBarcode]
GO
ALTER TABLE [dbo].[Forms] ADD  DEFAULT ((0)) FOR [IsLibrary]
GO
ALTER TABLE [dbo].[Logs] ADD  CONSTRAINT [DF_Logs_Id]  DEFAULT (newid()) FOR [Id]
GO
ALTER TABLE [dbo].[Menus] ADD  DEFAULT ((0)) FOR [DisableEn]
GO
ALTER TABLE [dbo].[Menus] ADD  DEFAULT ((0)) FOR [DisableAr]
GO
ALTER TABLE [dbo].[NotificationDefinitions] ADD  CONSTRAINT [DF_NotificationDefinitions_Id]  DEFAULT (newid()) FOR [Id]
GO
ALTER TABLE [dbo].[NotificationDefinitions] ADD  CONSTRAINT [DF_NotificationDefinitions_FrequencyIntervalRate]  DEFAULT ('M') FOR [FrequencyIntervalRate]
GO
ALTER TABLE [dbo].[NotificationDefinitions] ADD  CONSTRAINT [DF_NotificationDefinitions_NotificationType]  DEFAULT ('A') FOR [NotificationType]
GO
ALTER TABLE [dbo].[NotificationTransactions] ADD  CONSTRAINT [DF_NotificationTransactions_Id]  DEFAULT (newid()) FOR [Id]
GO
ALTER TABLE [dbo].[ProductOrders] ADD  DEFAULT ((0)) FOR [IsCopied]
GO
ALTER TABLE [dbo].[QueuedEmails] ADD  CONSTRAINT [DF_QueuedEmails_SentTries]  DEFAULT ((0)) FOR [SentTries]
GO
ALTER TABLE [dbo].[QueuedEmails] ADD  CONSTRAINT [DF_QueuedEmails_DateCreated]  DEFAULT (getdate()) FOR [DateCreated]
GO
ALTER TABLE [dbo].[QueuedEmails] ADD  CONSTRAINT [DF_QueuedEmails_IsDeleted]  DEFAULT ((0)) FOR [IsDeleted]
GO
ALTER TABLE [dbo].[Videos] ADD  DEFAULT ((0)) FOR [IsCompressed]
GO
ALTER TABLE [dbo].[Worklists] ADD  CONSTRAINT [DF_Worklist_Id]  DEFAULT (newid()) FOR [Id]
GO
ALTER TABLE [dbo].[AspNetUserClaims]  WITH CHECK ADD  CONSTRAINT [FK_dbo.AspNetUserClaims_dbo.AspNetUsers_UserId] FOREIGN KEY([UserId])
REFERENCES [dbo].[AspNetUsers] ([Id])
ON DELETE CASCADE
GO
ALTER TABLE [dbo].[AspNetUserClaims] CHECK CONSTRAINT [FK_dbo.AspNetUserClaims_dbo.AspNetUsers_UserId]
GO
ALTER TABLE [dbo].[AspNetUserLogins]  WITH CHECK ADD  CONSTRAINT [FK_dbo.AspNetUserLogins_dbo.AspNetUsers_UserId] FOREIGN KEY([UserId])
REFERENCES [dbo].[AspNetUsers] ([Id])
ON DELETE CASCADE
GO
ALTER TABLE [dbo].[AspNetUserLogins] CHECK CONSTRAINT [FK_dbo.AspNetUserLogins_dbo.AspNetUsers_UserId]
GO
ALTER TABLE [dbo].[AspNetUserRoles]  WITH CHECK ADD  CONSTRAINT [FK_dbo.AspNetUserRoles_dbo.AspNetRoles_RoleId] FOREIGN KEY([RoleId])
REFERENCES [dbo].[AspNetRoles] ([Id])
ON DELETE CASCADE
GO
ALTER TABLE [dbo].[AspNetUserRoles] CHECK CONSTRAINT [FK_dbo.AspNetUserRoles_dbo.AspNetRoles_RoleId]
GO
ALTER TABLE [dbo].[AspNetUserRoles]  WITH CHECK ADD  CONSTRAINT [FK_dbo.AspNetUserRoles_dbo.AspNetUsers_UserId] FOREIGN KEY([UserId])
REFERENCES [dbo].[AspNetUsers] ([Id])
ON DELETE CASCADE
GO
ALTER TABLE [dbo].[AspNetUserRoles] CHECK CONSTRAINT [FK_dbo.AspNetUserRoles_dbo.AspNetUsers_UserId]
GO
ALTER TABLE [dbo].[Categories]  WITH CHECK ADD  CONSTRAINT [FK_Categories_Categories] FOREIGN KEY([ParentId])
REFERENCES [dbo].[Categories] ([Id])
GO
ALTER TABLE [dbo].[Categories] CHECK CONSTRAINT [FK_Categories_Categories]
GO
ALTER TABLE [dbo].[Constants]  WITH CHECK ADD  CONSTRAINT [FK_Constants_ConstantGroups] FOREIGN KEY([ParentGroupId])
REFERENCES [dbo].[ConstantGroups] ([Id])
GO
ALTER TABLE [dbo].[Constants] CHECK CONSTRAINT [FK_Constants_ConstantGroups]
GO
ALTER TABLE [dbo].[Constants]  WITH CHECK ADD  CONSTRAINT [FK_Constants_Constants] FOREIGN KEY([ParentId])
REFERENCES [dbo].[Constants] ([Id])
GO
ALTER TABLE [dbo].[Constants] CHECK CONSTRAINT [FK_Constants_Constants]
GO
ALTER TABLE [dbo].[Contents]  WITH CHECK ADD  CONSTRAINT [FK_Contents_Languages] FOREIGN KEY([LanguageId])
REFERENCES [dbo].[Languages] ([Id])
GO
ALTER TABLE [dbo].[Contents] CHECK CONSTRAINT [FK_Contents_Languages]
GO
ALTER TABLE [dbo].[Contents]  WITH CHECK ADD  CONSTRAINT [FK_ic_Contents_ic_ContentTypes] FOREIGN KEY([ContentTypeId])
REFERENCES [dbo].[ContentTypes] ([Id])
GO
ALTER TABLE [dbo].[Contents] CHECK CONSTRAINT [FK_ic_Contents_ic_ContentTypes]
GO
ALTER TABLE [dbo].[EventExperts]  WITH CHECK ADD  CONSTRAINT [FK_EventExperts_Categories] FOREIGN KEY([ExpertCategoryId])
REFERENCES [dbo].[Categories] ([Id])
GO
ALTER TABLE [dbo].[EventExperts] CHECK CONSTRAINT [FK_EventExperts_Categories]
GO
ALTER TABLE [dbo].[EventExperts]  WITH CHECK ADD  CONSTRAINT [FK_EventExperts_Events] FOREIGN KEY([EventId])
REFERENCES [dbo].[Events] ([Id])
GO
ALTER TABLE [dbo].[EventExperts] CHECK CONSTRAINT [FK_EventExperts_Events]
GO
ALTER TABLE [dbo].[EventExperts]  WITH CHECK ADD  CONSTRAINT [FK_EventExperts_Experts] FOREIGN KEY([ExpertId])
REFERENCES [dbo].[Experts] ([Id])
GO
ALTER TABLE [dbo].[EventExperts] CHECK CONSTRAINT [FK_EventExperts_Experts]
GO
ALTER TABLE [dbo].[EventRegistrations]  WITH CHECK ADD  CONSTRAINT [FK_EventRegistrations_Events] FOREIGN KEY([EventId])
REFERENCES [dbo].[Events] ([Id])
GO
ALTER TABLE [dbo].[EventRegistrations] CHECK CONSTRAINT [FK_EventRegistrations_Events]
GO
ALTER TABLE [dbo].[Events]  WITH CHECK ADD  CONSTRAINT [FK_Events_Categories] FOREIGN KEY([CategoryId])
REFERENCES [dbo].[Categories] ([Id])
GO
ALTER TABLE [dbo].[Events] CHECK CONSTRAINT [FK_Events_Categories]
GO
ALTER TABLE [dbo].[Experts]  WITH CHECK ADD  CONSTRAINT [FK_Experts_Constants] FOREIGN KEY([CountryId])
REFERENCES [dbo].[Constants] ([Id])
GO
ALTER TABLE [dbo].[Experts] CHECK CONSTRAINT [FK_Experts_Constants]
GO
ALTER TABLE [dbo].[Forms]  WITH CHECK ADD  CONSTRAINT [FK_Forms_Categories] FOREIGN KEY([CategoryId])
REFERENCES [dbo].[Categories] ([Id])
GO
ALTER TABLE [dbo].[Forms] CHECK CONSTRAINT [FK_Forms_Categories]
GO
ALTER TABLE [dbo].[FormSubmissions]  WITH CHECK ADD  CONSTRAINT [FK_FormSubmissions_Forms] FOREIGN KEY([FormId])
REFERENCES [dbo].[Forms] ([Id])
GO
ALTER TABLE [dbo].[FormSubmissions] CHECK CONSTRAINT [FK_FormSubmissions_Forms]
GO
ALTER TABLE [dbo].[LibraryReservationSlots]  WITH CHECK ADD  CONSTRAINT [FK_LibraryReservationSlots_LibraryReservations] FOREIGN KEY([LibraryReservationId])
REFERENCES [dbo].[LibraryReservations] ([Id])
GO
ALTER TABLE [dbo].[LibraryReservationSlots] CHECK CONSTRAINT [FK_LibraryReservationSlots_LibraryReservations]
GO
ALTER TABLE [dbo].[Menus]  WITH CHECK ADD  CONSTRAINT [FK_Menus_Menus] FOREIGN KEY([Parent])
REFERENCES [dbo].[Menus] ([Id])
GO
ALTER TABLE [dbo].[Menus] CHECK CONSTRAINT [FK_Menus_Menus]
GO
ALTER TABLE [dbo].[News]  WITH CHECK ADD  CONSTRAINT [FK_News_Categories] FOREIGN KEY([CategoryId])
REFERENCES [dbo].[Categories] ([Id])
GO
ALTER TABLE [dbo].[News] CHECK CONSTRAINT [FK_News_Categories]
GO
ALTER TABLE [dbo].[NewsExperts]  WITH CHECK ADD  CONSTRAINT [FK_NewsExperts_Categories] FOREIGN KEY([ExpertCategoryId])
REFERENCES [dbo].[Categories] ([Id])
GO
ALTER TABLE [dbo].[NewsExperts] CHECK CONSTRAINT [FK_NewsExperts_Categories]
GO
ALTER TABLE [dbo].[NewsExperts]  WITH CHECK ADD  CONSTRAINT [FK_NewsExperts_Experts] FOREIGN KEY([ExpertId])
REFERENCES [dbo].[Experts] ([Id])
GO
ALTER TABLE [dbo].[NewsExperts] CHECK CONSTRAINT [FK_NewsExperts_Experts]
GO
ALTER TABLE [dbo].[NewsExperts]  WITH CHECK ADD  CONSTRAINT [FK_NewsExperts_News] FOREIGN KEY([NewsId])
REFERENCES [dbo].[News] ([Id])
GO
ALTER TABLE [dbo].[NewsExperts] CHECK CONSTRAINT [FK_NewsExperts_News]
GO
ALTER TABLE [dbo].[NewsLetterSubscription]  WITH CHECK ADD  CONSTRAINT [FK_NewsLetterSubscription_Languages] FOREIGN KEY([LanguageId])
REFERENCES [dbo].[Languages] ([Id])
GO
ALTER TABLE [dbo].[NewsLetterSubscription] CHECK CONSTRAINT [FK_NewsLetterSubscription_Languages]
GO
ALTER TABLE [dbo].[NotificationTransactions]  WITH CHECK ADD  CONSTRAINT [FK_NotificationTransactions_NotificationDefinitions] FOREIGN KEY([NotificationDefinitionId])
REFERENCES [dbo].[NotificationDefinitions] ([Id])
GO
ALTER TABLE [dbo].[NotificationTransactions] CHECK CONSTRAINT [FK_NotificationTransactions_NotificationDefinitions]
GO
ALTER TABLE [dbo].[ProductExperts]  WITH CHECK ADD  CONSTRAINT [FK_ProductExperts_Experts] FOREIGN KEY([ExpertId])
REFERENCES [dbo].[Experts] ([Id])
GO
ALTER TABLE [dbo].[ProductExperts] CHECK CONSTRAINT [FK_ProductExperts_Experts]
GO
ALTER TABLE [dbo].[ProductExperts]  WITH CHECK ADD  CONSTRAINT [FK_ProductExperts_Products] FOREIGN KEY([ProductId])
REFERENCES [dbo].[Products] ([Id])
GO
ALTER TABLE [dbo].[ProductExperts] CHECK CONSTRAINT [FK_ProductExperts_Products]
GO
ALTER TABLE [dbo].[ProductOptions]  WITH CHECK ADD  CONSTRAINT [FK_ProductDetails_Products] FOREIGN KEY([ProductId])
REFERENCES [dbo].[Products] ([Id])
ON UPDATE CASCADE
ON DELETE CASCADE
GO
ALTER TABLE [dbo].[ProductOptions] CHECK CONSTRAINT [FK_ProductDetails_Products]
GO
ALTER TABLE [dbo].[ProductOptions]  WITH CHECK ADD  CONSTRAINT [FK_ProductOptions_Langauges] FOREIGN KEY([LanguageId])
REFERENCES [dbo].[Languages] ([Id])
GO
ALTER TABLE [dbo].[ProductOptions] CHECK CONSTRAINT [FK_ProductOptions_Langauges]
GO
ALTER TABLE [dbo].[ProductOrderItems]  WITH CHECK ADD  CONSTRAINT [FK_ECommerceProductOrderItems_ECommerceProductOrders] FOREIGN KEY([ProductOrderId])
REFERENCES [dbo].[ProductOrders] ([Id])
GO
ALTER TABLE [dbo].[ProductOrderItems] CHECK CONSTRAINT [FK_ECommerceProductOrderItems_ECommerceProductOrders]
GO
ALTER TABLE [dbo].[ProductOrderItems]  WITH CHECK ADD  CONSTRAINT [FK_ECommerceProductOrderItems_ECommerceProducts] FOREIGN KEY([ProductId])
REFERENCES [dbo].[Products] ([Id])
GO
ALTER TABLE [dbo].[ProductOrderItems] CHECK CONSTRAINT [FK_ECommerceProductOrderItems_ECommerceProducts]
GO
ALTER TABLE [dbo].[ProductOrderItems]  WITH CHECK ADD  CONSTRAINT [FK_ProductOrderItems_ProductOptions] FOREIGN KEY([ProductOptionId])
REFERENCES [dbo].[ProductOptions] ([Id])
GO
ALTER TABLE [dbo].[ProductOrderItems] CHECK CONSTRAINT [FK_ProductOrderItems_ProductOptions]
GO
ALTER TABLE [dbo].[ProductOrders]  WITH CHECK ADD  CONSTRAINT [FK_ProductOrders_AspNetUsers] FOREIGN KEY([UserId])
REFERENCES [dbo].[AspNetUsers] ([Id])
GO
ALTER TABLE [dbo].[ProductOrders] CHECK CONSTRAINT [FK_ProductOrders_AspNetUsers]
GO
ALTER TABLE [dbo].[ProductOrders]  WITH CHECK ADD  CONSTRAINT [FK_ProductOrders_UserShippingAddresses] FOREIGN KEY([UserShippingAddressId])
REFERENCES [dbo].[UserShippingAddresses] ([Id])
GO
ALTER TABLE [dbo].[ProductOrders] CHECK CONSTRAINT [FK_ProductOrders_UserShippingAddresses]
GO
ALTER TABLE [dbo].[ProductOrderTransactions]  WITH CHECK ADD  CONSTRAINT [FK_ProductOrderTransactions_ProductOrders] FOREIGN KEY([ProductOrderId])
REFERENCES [dbo].[ProductOrders] ([Id])
GO
ALTER TABLE [dbo].[ProductOrderTransactions] CHECK CONSTRAINT [FK_ProductOrderTransactions_ProductOrders]
GO
ALTER TABLE [dbo].[Products]  WITH CHECK ADD  CONSTRAINT [FK_Products_Categories] FOREIGN KEY([CategoryId])
REFERENCES [dbo].[Categories] ([Id])
GO
ALTER TABLE [dbo].[Products] CHECK CONSTRAINT [FK_Products_Categories]
GO
ALTER TABLE [dbo].[Products]  WITH CHECK ADD  CONSTRAINT [FK_Products_Categories_Sub] FOREIGN KEY([SubCategoryId])
REFERENCES [dbo].[Categories] ([Id])
GO
ALTER TABLE [dbo].[Products] CHECK CONSTRAINT [FK_Products_Categories_Sub]
GO
ALTER TABLE [dbo].[ProductStockNotifications]  WITH CHECK ADD  CONSTRAINT [FK_ProductStockNotifications_ProductOptions] FOREIGN KEY([ProductOptionId])
REFERENCES [dbo].[ProductOptions] ([Id])
GO
ALTER TABLE [dbo].[ProductStockNotifications] CHECK CONSTRAINT [FK_ProductStockNotifications_ProductOptions]
GO
ALTER TABLE [dbo].[ProductStockNotifications]  WITH CHECK ADD  CONSTRAINT [FK_ProductStockNotifications_Products] FOREIGN KEY([ProductId])
REFERENCES [dbo].[Products] ([Id])
GO
ALTER TABLE [dbo].[ProductStockNotifications] CHECK CONSTRAINT [FK_ProductStockNotifications_Products]
GO
ALTER TABLE [dbo].[SSR]  WITH CHECK ADD  CONSTRAINT [FK_SSR_Categories] FOREIGN KEY([CategoryId])
REFERENCES [dbo].[Categories] ([Id])
GO
ALTER TABLE [dbo].[SSR] CHECK CONSTRAINT [FK_SSR_Categories]
GO
ALTER TABLE [dbo].[StrategicPartners]  WITH CHECK ADD  CONSTRAINT [FK_StrategicPartners_Constants] FOREIGN KEY([CountryId])
REFERENCES [dbo].[Constants] ([Id])
GO
ALTER TABLE [dbo].[StrategicPartners] CHECK CONSTRAINT [FK_StrategicPartners_Constants]
GO
ALTER TABLE [dbo].[UserFavorites]  WITH CHECK ADD  CONSTRAINT [FK_UserFavorites_AspNetUsers] FOREIGN KEY([UserId])
REFERENCES [dbo].[AspNetUsers] ([Id])
GO
ALTER TABLE [dbo].[UserFavorites] CHECK CONSTRAINT [FK_UserFavorites_AspNetUsers]
GO
ALTER TABLE [dbo].[UserProducts]  WITH CHECK ADD  CONSTRAINT [FK_UserProducts_AspNetUsers] FOREIGN KEY([UserId])
REFERENCES [dbo].[AspNetUsers] ([Id])
GO
ALTER TABLE [dbo].[UserProducts] CHECK CONSTRAINT [FK_UserProducts_AspNetUsers]
GO
ALTER TABLE [dbo].[UserProducts]  WITH CHECK ADD  CONSTRAINT [FK_UserProducts_Product] FOREIGN KEY([ProductId])
REFERENCES [dbo].[Products] ([Id])
GO
ALTER TABLE [dbo].[UserProducts] CHECK CONSTRAINT [FK_UserProducts_Product]
GO
ALTER TABLE [dbo].[UserProducts]  WITH CHECK ADD  CONSTRAINT [FK_UserProducts_ProductOptions] FOREIGN KEY([ProductOptionId])
REFERENCES [dbo].[ProductOptions] ([Id])
GO
ALTER TABLE [dbo].[UserProducts] CHECK CONSTRAINT [FK_UserProducts_ProductOptions]
GO
ALTER TABLE [dbo].[UserProducts]  WITH CHECK ADD  CONSTRAINT [FK_UserProducts_ProductOrders] FOREIGN KEY([ProductOrderId])
REFERENCES [dbo].[ProductOrders] ([Id])
GO
ALTER TABLE [dbo].[UserProducts] CHECK CONSTRAINT [FK_UserProducts_ProductOrders]
GO
ALTER TABLE [dbo].[UserShippingAddresses]  WITH CHECK ADD  CONSTRAINT [FK_UserShippingAddresses_AspNetUsers] FOREIGN KEY([UserId])
REFERENCES [dbo].[AspNetUsers] ([Id])
GO
ALTER TABLE [dbo].[UserShippingAddresses] CHECK CONSTRAINT [FK_UserShippingAddresses_AspNetUsers]
GO
ALTER TABLE [dbo].[Videos]  WITH CHECK ADD  CONSTRAINT [FK_Videos_Categories] FOREIGN KEY([CategoryId])
REFERENCES [dbo].[Categories] ([Id])
GO
ALTER TABLE [dbo].[Videos] CHECK CONSTRAINT [FK_Videos_Categories]
GO
ALTER TABLE [HangFire].[JobParameter]  WITH CHECK ADD  CONSTRAINT [FK_HangFire_JobParameter_Job] FOREIGN KEY([JobId])
REFERENCES [HangFire].[Job] ([Id])
ON UPDATE CASCADE
ON DELETE CASCADE
GO
ALTER TABLE [HangFire].[JobParameter] CHECK CONSTRAINT [FK_HangFire_JobParameter_Job]
GO
ALTER TABLE [HangFire].[State]  WITH CHECK ADD  CONSTRAINT [FK_HangFire_State_Job] FOREIGN KEY([JobId])
REFERENCES [HangFire].[Job] ([Id])
ON UPDATE CASCADE
ON DELETE CASCADE
GO
ALTER TABLE [HangFire].[State] CHECK CONSTRAINT [FK_HangFire_State_Job]
GO
/****** Object:  StoredProcedure [dbo].[ContentSearch]    Script Date: 3/11/2024 8:29:35 AM ******/
SET ANSI_NULLS ON
GO
SET QUOTED_IDENTIFIER ON
GO

CREATE PROCEDURE [dbo].[ContentSearch]  
(
	@contentTypeId	NVARCHAR(50) = NULL	,
	/*– Pagination Parameters */
	@pageIndex int=1,
    @pageSize int=10,

	/*– Sorting Parameters */
	@SortColumn NVARCHAR(256) = 'LastUpdateDate Desc',
	@extraSelect NVARCHAR(max),
    @extraSearchParams NVARCHAR(max) 
)
AS          
BEGIN      
	SET NOCOUNT ON;  
 
	DECLARE @SQL							NVARCHAR(MAX)
	DECLARE @ParameterDef					NVARCHAR(500)
 
    SET @ParameterDef =    
	' @contentTypeId			NVARCHAR(50) = NULL	,
	  /*– Pagination Parameters */
	 @pageIndex int=1,
		@pageSize int=10,

	/*– Sorting Parameters */
	@SortColumn NVARCHAR(30) = ''LastUpdateDate Desc'''

	
 
 
    SET @SQL = 'DECLARE @lPageNbr INT,
    @lPageSize INT,
    @lFirstRec INT,
    @lLastRec INT,
    @lTotalRows INT

	SET @lPageNbr = @pageIndex
    SET @lPageSize = @PageSize

    SET @lFirstRec = ( @lPageNbr - 1 ) * @lPageSize
    SET @lLastRec = ( @lPageNbr * @lPageSize + 1 )
    SET @lTotalRows = @lFirstRec - @lLastRec + 1
  ; WITH Results
			AS (
				SELECT ROW_NUMBER() OVER (ORDER BY '+LTRIM(RTRIM(@SortColumn))+'
			  ) AS RowNumber,
			  Count(*) over () AS TotalCount,
			  * from (select
			  Contents.Id,
			  Contents.ContentTypeId,
			  Contents.IsLocked,
			  Contents.IsRecurring,
			  Contents.IsSticky,
			  Contents.MetaPageTitle,
		      Contents.MetaKeywords,
		      Contents.MetaDescription,
			  Contents.LockedBy,
			  Contents.LockDate,
			  Contents.Title,
			  Contents.IntroText,
			  Contents.FullText,
			  Contents.Slug,
			  Contents.Images,
			  Contents.Attributes,
			  Contents.RelatedItems,
			  (case Contents.IsRecurring when 1 then  DATEADD(YEAR,YEAR(GETUTCDATE()) - YEAR(Contents.ItemDate1),Contents.ItemDate1)  else Contents.ItemDate1 end) as ItemDate1,
			  (case Contents.IsRecurring when 1 then  DATEADD(YEAR,YEAR(GETUTCDATE()) - YEAR(Contents.ItemDate2),Contents.ItemDate2)  else Contents.ItemDate2 end) as ItemDate2,
			  Contents.IsPublished,
			  Contents.IsDeleted,
			  Contents.DateCreated,
			  Contents.LastUpdateDate,
			  Contents.CreatedBy,
			  Contents.LastUpdatedBy,
			  Contents.Likes,
			  Contents.Dislikes,
			  Contents.IsFullyTranslated,
			  Contents.CategoriesIds,
			  Contents.ContentThemeIds,
			  Contents.FullUrl, 
              Contents.ViewsCount,
			  Contents.WorkflowStatus,
			  Contents.WorkflowSummary,
			  Contents.LanguageId, 
			  Contents.LocalizationGroupId,
			  dbo.[GetContentStatus](Contents.IsPublished,Contents.IsArchived,Contents.IsDeleted,Contents.PublishDateFrom,Contents.PublishDateTo,Contents.ArchiveDateFrom,Contents.ArchiveDateTo) as Status,
			  ct.Type
			  '+@extraSelect+'
			 FROM dbo.Contents 
				left join ContentTypes ct on  ct.Id=ContentTypeId
		     	where (@contentTypeId IS NULL OR ContentTypeId = @contentTypeId)
			) as content 
			 WHERE
				(@contentTypeId IS NULL OR ContentTypeId = @contentTypeId)
				'+@extraSearchParams+'
		  	)
			SELECT
				*
			FROM Results
			WHERE
					RowNumber > @lFirstRec
						  AND RowNumber < @lLastRec
			 ORDER BY RowNumber ASC' 
  
  print @SQL

   EXEC sp_Executesql     @SQL,  @ParameterDef, @contentTypeId=@contentTypeId,@pageIndex=@pageIndex,@pageSize=@pageSize,@SortColumn=@SortColumn
               
END
 
-------------------------------------------------------------------------
 
GO
/****** Object:  StoredProcedure [dbo].[FormSubmissionsSearch]    Script Date: 3/11/2024 8:29:35 AM ******/
SET ANSI_NULLS ON
GO
SET QUOTED_IDENTIFIER ON
GO
CREATE    PROCEDURE [dbo].[FormSubmissionsSearch]  
(
    /*– Pagination Parameters */
    @pageIndex int=1,
    @pageSize int=10,     /*– Sorting Parameters */
    @SortColumn NVARCHAR(256) = 'LastUpdateDate Desc',
    @extraSelect NVARCHAR(max)='',
    @extraSearchParams NVARCHAR(max)='' )
AS          
BEGIN      
    SET NOCOUNT ON;  
    DECLARE @SQL                            NVARCHAR(MAX)
    DECLARE @ParameterDef                    NVARCHAR(500)
    SET @ParameterDef =    
    ' /*– Pagination Parameters */
     @pageIndex int=1,
        @pageSize int=10,     /*– Sorting Parameters */
    @SortColumn NVARCHAR(30) = ''LastUpdateDate Desc'''     
    SET @SQL = 'DECLARE @lPageNbr INT,
    @lPageSize INT,
    @lFirstRec INT,
    @lLastRec INT,
    @lTotalRows INT     SET @lPageNbr = @pageIndex
    SET @lPageSize = @PageSize     SET @lFirstRec = ( @lPageNbr - 1 ) * @lPageSize
    SET @lLastRec = ( @lPageNbr * @lPageSize + 1 )
    SET @lTotalRows = @lFirstRec - @lLastRec + 1
  ; WITH Results
            AS (
                SELECT ROW_NUMBER() OVER (ORDER BY '+LTRIM(RTRIM(@SortColumn))+'
              ) AS RowNumber,
              Count(*) over () AS TotalCount,
              * from (select
              FormSubmissions.Id,
              FormSubmissions.FormId,
              FormSubmissions.RefNumber,
              FormSubmissions.UserAgent,
              FormSubmissions.IPAddress,
              FormSubmissions.FormValues,
              FormSubmissions.IsPublished,
              FormSubmissions.IsDeleted,
              FormSubmissions.IsRead,
              FormSubmissions.LanguageId,
              FormSubmissions.DateCreated,
              FormSubmissions.LastUpdateDate,
              FormSubmissions.CreatedBy,
              FormSubmissions.WorkflowStatus,
              FormSubmissions.WorkflowSummary,
              FormSubmissions.LastUpdatedBy
              '+@extraSelect+'
             FROM dbo.FormSubmissions             
            ) as FormSubmission 
             WHERE
                (1=1)
                '+@extraSearchParams+'
              )
            SELECT
                *
            FROM Results
            WHERE
                    RowNumber > @lFirstRec
                          AND RowNumber < @lLastRec
             ORDER BY RowNumber ASC' 
  print @SQL    EXEC sp_Executesql     @SQL,  @ParameterDef,@pageIndex=@pageIndex,@pageSize=@pageSize,@SortColumn=@SortColumn
END

GO
/****** Object:  StoredProcedure [dbo].[GetEntityDescendants]    Script Date: 3/11/2024 8:29:35 AM ******/
SET ANSI_NULLS ON
GO
SET QUOTED_IDENTIFIER ON
GO

CREATE PROCEDURE [dbo].[GetEntityDescendants]
    @Id  uniqueidentifier
AS


BEGIN
   
	DECLARE @hierarchy hierarchyID;

	SELECT @hierarchy = Hierarchy FROM EntitiesHierarchy WHERE Id = @Id
	SELECT Id FROM EntitiesHierarchy WHERE Hierarchy.IsDescendantOf(@hierarchy) = 1

END

GO
/****** Object:  StoredProcedure [dbo].[GetWorkingDaysCount]    Script Date: 3/11/2024 8:29:35 AM ******/
SET ANSI_NULLS ON
GO
SET QUOTED_IDENTIFIER ON
GO

CREATE PROCEDURE [dbo].[GetWorkingDaysCount] --'2022-02-27','2022-03-12'
	@startDate datetime,
	@endDate datetime
AS
BEGIN
	-- SET NOCOUNT ON added to prevent extra result sets from
	-- interfering with SELECT statements.
	SET NOCOUNT ON;
	DECLARE @CurrentDate AS DATETIME
	DECLARE @Count AS INT
	
	SET @CurrentDate = @startDate
	SET @Count = 0

	WHILE (@CurrentDate <= @endDate)
	BEGIN
		DECLARE @DayName NVARCHAR(50)
		SELECT @DayName = UPPER(DATENAME(WEEKDAY, @CurrentDate));
		
		IF EXISTS (SELECT 1 FROM WorkingDays WHERE UPPER(DayName) = @DayName AND IsDeleted= 0)
			BEGIN
				IF NOT EXISTS (SELECT 1 FROM Vacations WHERE StartDate <= @CurrentDate AND EndDate >= @CurrentDate AND IsDeleted=0)
				BEGIN
				   SET @Count =@Count+1
				   print @DayName
				END
			END
		SET @CurrentDate = DATEADD(DAY, 1, @CurrentDate); /*increment current date*/	
	END
	SELECT @Count AS WorkingDays

END
GO
/****** Object:  StoredProcedure [dbo].[IsUserInEntity]    Script Date: 3/11/2024 8:29:35 AM ******/
SET ANSI_NULLS ON
GO
SET QUOTED_IDENTIFIER ON
GO

CREATE PROCEDURE [dbo].[IsUserInEntity]
    @entityId  uniqueidentifier,
	@userId nvarchar(128)
AS


BEGIN
	DECLARE @hierarchy hierarchyID;
	SELECT @hierarchy = Hierarchy FROM EntitiesHierarchy WHERE Id = @entityId;	
	SELECT  count(*) from dbo.AspNetUsers where Id = @userId and EntityId in (SELECT Id FROM EntitiesHierarchy WHERE Hierarchy.IsDescendantOf(@hierarchy) = 1);
END
GO
EXEC sys.sp_addextendedproperty @name=N'MS_Description', @value=N'''A'' for admin menu , ''M'' for main menu' , @level0type=N'SCHEMA',@level0name=N'dbo', @level1type=N'TABLE',@level1name=N'Menus', @level2type=N'COLUMN',@level2name=N'MenuType'
GO
EXEC sys.sp_addextendedproperty @name=N'MS_Description', @value=N'''M'' menu , ''P''  page' , @level0type=N'SCHEMA',@level0name=N'dbo', @level1type=N'TABLE',@level1name=N'Menus', @level2type=N'COLUMN',@level2name=N'Type'
GO
EXEC sys.sp_addextendedproperty @name=N'MS_Description', @value=N'''B'' _blank, ''S'' _self, ''P'' _parent, ''T'' _top' , @level0type=N'SCHEMA',@level0name=N'dbo', @level1type=N'TABLE',@level1name=N'Menus', @level2type=N'COLUMN',@level2name=N'Target'
GO
EXEC sys.sp_addextendedproperty @name=N'MS_Description', @value=N'S For soft, H for hard' , @level0type=N'SCHEMA',@level0name=N'dbo', @level1type=N'TABLE',@level1name=N'ProductOptions', @level2type=N'COLUMN',@level2name=N'ProductType'
GO
EXEC sys.sp_addextendedproperty @name=N'MS_DiagramPane1', @value=N'[0E232FF0-B466-11cf-A24F-00AA00A3EFFF, 1.00]
Begin DesignProperties = 
   Begin PaneConfigurations = 
      Begin PaneConfiguration = 0
         NumPanes = 4
         Configuration = "(H (1[40] 4[20] 2[20] 3) )"
      End
      Begin PaneConfiguration = 1
         NumPanes = 3
         Configuration = "(H (1 [50] 4 [25] 3))"
      End
      Begin PaneConfiguration = 2
         NumPanes = 3
         Configuration = "(H (1 [50] 2 [25] 3))"
      End
      Begin PaneConfiguration = 3
         NumPanes = 3
         Configuration = "(H (4 [30] 2 [40] 3))"
      End
      Begin PaneConfiguration = 4
         NumPanes = 2
         Configuration = "(H (1 [56] 3))"
      End
      Begin PaneConfiguration = 5
         NumPanes = 2
         Configuration = "(H (2 [66] 3))"
      End
      Begin PaneConfiguration = 6
         NumPanes = 2
         Configuration = "(H (4 [50] 3))"
      End
      Begin PaneConfiguration = 7
         NumPanes = 1
         Configuration = "(V (3))"
      End
      Begin PaneConfiguration = 8
         NumPanes = 3
         Configuration = "(H (1[56] 4[18] 2) )"
      End
      Begin PaneConfiguration = 9
         NumPanes = 2
         Configuration = "(H (1 [75] 4))"
      End
      Begin PaneConfiguration = 10
         NumPanes = 2
         Configuration = "(H (1[66] 2) )"
      End
      Begin PaneConfiguration = 11
         NumPanes = 2
         Configuration = "(H (4 [60] 2))"
      End
      Begin PaneConfiguration = 12
         NumPanes = 1
         Configuration = "(H (1) )"
      End
      Begin PaneConfiguration = 13
         NumPanes = 1
         Configuration = "(V (4))"
      End
      Begin PaneConfiguration = 14
         NumPanes = 1
         Configuration = "(V (2))"
      End
      ActivePaneConfig = 0
   End
   Begin DiagramPane = 
      Begin Origin = 
         Top = 0
         Left = 0
      End
      Begin Tables = 
         Begin Table = "Announcements"
            Begin Extent = 
               Top = 6
               Left = 38
               Bottom = 282
               Right = 208
            End
            DisplayFlags = 280
            TopColumn = 0
         End
         Begin Table = "Translations"
            Begin Extent = 
               Top = 6
               Left = 246
               Bottom = 136
               Right = 430
            End
            DisplayFlags = 280
            TopColumn = 0
         End
      End
   End
   Begin SQLPane = 
   End
   Begin DataPane = 
      Begin ParameterDefaults = ""
      End
   End
   Begin CriteriaPane = 
      Begin ColumnWidths = 11
         Column = 1440
         Alias = 900
         Table = 1170
         Output = 720
         Append = 1400
         NewValue = 1170
         SortType = 1350
         SortOrder = 1410
         GroupBy = 1350
         Filter = 1350
         Or = 1350
         Or = 1350
         Or = 1350
      End
   End
End
' , @level0type=N'SCHEMA',@level0name=N'dbo', @level1type=N'VIEW',@level1name=N'TranslatedAnnouncements'
GO
EXEC sys.sp_addextendedproperty @name=N'MS_DiagramPaneCount', @value=1 , @level0type=N'SCHEMA',@level0name=N'dbo', @level1type=N'VIEW',@level1name=N'TranslatedAnnouncements'
GO
EXEC sys.sp_addextendedproperty @name=N'MS_DiagramPane1', @value=N'[0E232FF0-B466-11cf-A24F-00AA00A3EFFF, 1.00]
Begin DesignProperties = 
   Begin PaneConfigurations = 
      Begin PaneConfiguration = 0
         NumPanes = 4
         Configuration = "(H (1[40] 4[20] 2[20] 3) )"
      End
      Begin PaneConfiguration = 1
         NumPanes = 3
         Configuration = "(H (1 [50] 4 [25] 3))"
      End
      Begin PaneConfiguration = 2
         NumPanes = 3
         Configuration = "(H (1 [50] 2 [25] 3))"
      End
      Begin PaneConfiguration = 3
         NumPanes = 3
         Configuration = "(H (4 [30] 2 [40] 3))"
      End
      Begin PaneConfiguration = 4
         NumPanes = 2
         Configuration = "(H (1 [56] 3))"
      End
      Begin PaneConfiguration = 5
         NumPanes = 2
         Configuration = "(H (2 [66] 3))"
      End
      Begin PaneConfiguration = 6
         NumPanes = 2
         Configuration = "(H (4 [50] 3))"
      End
      Begin PaneConfiguration = 7
         NumPanes = 1
         Configuration = "(V (3))"
      End
      Begin PaneConfiguration = 8
         NumPanes = 3
         Configuration = "(H (1[56] 4[18] 2) )"
      End
      Begin PaneConfiguration = 9
         NumPanes = 2
         Configuration = "(H (1 [75] 4))"
      End
      Begin PaneConfiguration = 10
         NumPanes = 2
         Configuration = "(H (1[66] 2) )"
      End
      Begin PaneConfiguration = 11
         NumPanes = 2
         Configuration = "(H (4 [60] 2))"
      End
      Begin PaneConfiguration = 12
         NumPanes = 1
         Configuration = "(H (1) )"
      End
      Begin PaneConfiguration = 13
         NumPanes = 1
         Configuration = "(V (4))"
      End
      Begin PaneConfiguration = 14
         NumPanes = 1
         Configuration = "(V (2))"
      End
      ActivePaneConfig = 0
   End
   Begin DiagramPane = 
      Begin Origin = 
         Top = 0
         Left = 0
      End
      Begin Tables = 
         Begin Table = "Banners"
            Begin Extent = 
               Top = 6
               Left = 38
               Bottom = 317
               Right = 208
            End
            DisplayFlags = 280
            TopColumn = 0
         End
         Begin Table = "Translations"
            Begin Extent = 
               Top = 6
               Left = 246
               Bottom = 282
               Right = 430
            End
            DisplayFlags = 280
            TopColumn = 0
         End
      End
   End
   Begin SQLPane = 
   End
   Begin DataPane = 
      Begin ParameterDefaults = ""
      End
   End
   Begin CriteriaPane = 
      Begin ColumnWidths = 11
         Column = 1440
         Alias = 900
         Table = 1170
         Output = 720
         Append = 1400
         NewValue = 1170
         SortType = 1350
         SortOrder = 1410
         GroupBy = 1350
         Filter = 1350
         Or = 1350
         Or = 1350
         Or = 1350
      End
   End
End
' , @level0type=N'SCHEMA',@level0name=N'dbo', @level1type=N'VIEW',@level1name=N'TranslatedBanners'
GO
EXEC sys.sp_addextendedproperty @name=N'MS_DiagramPaneCount', @value=1 , @level0type=N'SCHEMA',@level0name=N'dbo', @level1type=N'VIEW',@level1name=N'TranslatedBanners'
GO
EXEC sys.sp_addextendedproperty @name=N'MS_DiagramPane1', @value=N'[0E232FF0-B466-11cf-A24F-00AA00A3EFFF, 1.00]
Begin DesignProperties = 
   Begin PaneConfigurations = 
      Begin PaneConfiguration = 0
         NumPanes = 4
         Configuration = "(H (1[31] 4[20] 2[22] 3) )"
      End
      Begin PaneConfiguration = 1
         NumPanes = 3
         Configuration = "(H (1 [50] 4 [25] 3))"
      End
      Begin PaneConfiguration = 2
         NumPanes = 3
         Configuration = "(H (1 [50] 2 [25] 3))"
      End
      Begin PaneConfiguration = 3
         NumPanes = 3
         Configuration = "(H (4 [30] 2 [40] 3))"
      End
      Begin PaneConfiguration = 4
         NumPanes = 2
         Configuration = "(H (1 [56] 3))"
      End
      Begin PaneConfiguration = 5
         NumPanes = 2
         Configuration = "(H (2 [66] 3))"
      End
      Begin PaneConfiguration = 6
         NumPanes = 2
         Configuration = "(H (4 [50] 3))"
      End
      Begin PaneConfiguration = 7
         NumPanes = 1
         Configuration = "(V (3))"
      End
      Begin PaneConfiguration = 8
         NumPanes = 3
         Configuration = "(H (1[56] 4[18] 2) )"
      End
      Begin PaneConfiguration = 9
         NumPanes = 2
         Configuration = "(H (1 [75] 4))"
      End
      Begin PaneConfiguration = 10
         NumPanes = 2
         Configuration = "(H (1[66] 2) )"
      End
      Begin PaneConfiguration = 11
         NumPanes = 2
         Configuration = "(H (4 [60] 2))"
      End
      Begin PaneConfiguration = 12
         NumPanes = 1
         Configuration = "(H (1) )"
      End
      Begin PaneConfiguration = 13
         NumPanes = 1
         Configuration = "(V (4))"
      End
      Begin PaneConfiguration = 14
         NumPanes = 1
         Configuration = "(V (2))"
      End
      ActivePaneConfig = 0
   End
   Begin DiagramPane = 
      Begin Origin = 
         Top = 0
         Left = 0
      End
      Begin Tables = 
         Begin Table = "Constants"
            Begin Extent = 
               Top = 6
               Left = 38
               Bottom = 250
               Right = 208
            End
            DisplayFlags = 280
            TopColumn = 0
         End
         Begin Table = "Translations"
            Begin Extent = 
               Top = 6
               Left = 246
               Bottom = 264
               Right = 430
            End
            DisplayFlags = 280
            TopColumn = 0
         End
      End
   End
   Begin SQLPane = 
   End
   Begin DataPane = 
      Begin ParameterDefaults = ""
      End
      Begin ColumnWidths = 10
         Width = 284
         Width = 1500
         Width = 1500
         Width = 1500
         Width = 2025
         Width = 1500
         Width = 1500
         Width = 1500
         Width = 1500
         Width = 2175
      End
   End
   Begin CriteriaPane = 
      Begin ColumnWidths = 11
         Column = 1440
         Alias = 900
         Table = 1170
         Output = 720
         Append = 1400
         NewValue = 1170
         SortType = 1350
         SortOrder = 1410
         GroupBy = 1350
         Filter = 1350
         Or = 1350
         Or = 1350
         Or = 1350
      End
   End
End
' , @level0type=N'SCHEMA',@level0name=N'dbo', @level1type=N'VIEW',@level1name=N'TranslatedConstants'
GO
EXEC sys.sp_addextendedproperty @name=N'MS_DiagramPaneCount', @value=1 , @level0type=N'SCHEMA',@level0name=N'dbo', @level1type=N'VIEW',@level1name=N'TranslatedConstants'
GO
EXEC sys.sp_addextendedproperty @name=N'MS_DiagramPane1', @value=N'[0E232FF0-B466-11cf-A24F-00AA00A3EFFF, 1.00]
Begin DesignProperties = 
   Begin PaneConfigurations = 
      Begin PaneConfiguration = 0
         NumPanes = 4
         Configuration = "(H (1[40] 4[20] 2[20] 3) )"
      End
      Begin PaneConfiguration = 1
         NumPanes = 3
         Configuration = "(H (1 [50] 4 [25] 3))"
      End
      Begin PaneConfiguration = 2
         NumPanes = 3
         Configuration = "(H (1 [50] 2 [25] 3))"
      End
      Begin PaneConfiguration = 3
         NumPanes = 3
         Configuration = "(H (4 [30] 2 [40] 3))"
      End
      Begin PaneConfiguration = 4
         NumPanes = 2
         Configuration = "(H (1 [56] 3))"
      End
      Begin PaneConfiguration = 5
         NumPanes = 2
         Configuration = "(H (2 [66] 3))"
      End
      Begin PaneConfiguration = 6
         NumPanes = 2
         Configuration = "(H (4 [50] 3))"
      End
      Begin PaneConfiguration = 7
         NumPanes = 1
         Configuration = "(V (3))"
      End
      Begin PaneConfiguration = 8
         NumPanes = 3
         Configuration = "(H (1[56] 4[18] 2) )"
      End
      Begin PaneConfiguration = 9
         NumPanes = 2
         Configuration = "(H (1 [75] 4))"
      End
      Begin PaneConfiguration = 10
         NumPanes = 2
         Configuration = "(H (1[66] 2) )"
      End
      Begin PaneConfiguration = 11
         NumPanes = 2
         Configuration = "(H (4 [60] 2))"
      End
      Begin PaneConfiguration = 12
         NumPanes = 1
         Configuration = "(H (1) )"
      End
      Begin PaneConfiguration = 13
         NumPanes = 1
         Configuration = "(V (4))"
      End
      Begin PaneConfiguration = 14
         NumPanes = 1
         Configuration = "(V (2))"
      End
      ActivePaneConfig = 0
   End
   Begin DiagramPane = 
      Begin Origin = 
         Top = 0
         Left = 0
      End
      Begin Tables = 
         Begin Table = "ContentThemes"
            Begin Extent = 
               Top = 6
               Left = 38
               Bottom = 234
               Right = 208
            End
            DisplayFlags = 280
            TopColumn = 0
         End
         Begin Table = "Translations"
            Begin Extent = 
               Top = 6
               Left = 246
               Bottom = 264
               Right = 430
            End
            DisplayFlags = 280
            TopColumn = 0
         End
      End
   End
   Begin SQLPane = 
   End
   Begin DataPane = 
      Begin ParameterDefaults = ""
      End
   End
   Begin CriteriaPane = 
      Begin ColumnWidths = 11
         Column = 1440
         Alias = 900
         Table = 1170
         Output = 720
         Append = 1400
         NewValue = 1170
         SortType = 1350
         SortOrder = 1410
         GroupBy = 1350
         Filter = 1350
         Or = 1350
         Or = 1350
         Or = 1350
      End
   End
End
' , @level0type=N'SCHEMA',@level0name=N'dbo', @level1type=N'VIEW',@level1name=N'TranslatedContentThemes'
GO
EXEC sys.sp_addextendedproperty @name=N'MS_DiagramPaneCount', @value=1 , @level0type=N'SCHEMA',@level0name=N'dbo', @level1type=N'VIEW',@level1name=N'TranslatedContentThemes'
GO
EXEC sys.sp_addextendedproperty @name=N'MS_DiagramPane1', @value=N'[0E232FF0-B466-11cf-A24F-00AA00A3EFFF, 1.00]
Begin DesignProperties = 
   Begin PaneConfigurations = 
      Begin PaneConfiguration = 0
         NumPanes = 4
         Configuration = "(H (1[40] 4[20] 2[20] 3) )"
      End
      Begin PaneConfiguration = 1
         NumPanes = 3
         Configuration = "(H (1 [50] 4 [25] 3))"
      End
      Begin PaneConfiguration = 2
         NumPanes = 3
         Configuration = "(H (1 [50] 2 [25] 3))"
      End
      Begin PaneConfiguration = 3
         NumPanes = 3
         Configuration = "(H (4 [30] 2 [40] 3))"
      End
      Begin PaneConfiguration = 4
         NumPanes = 2
         Configuration = "(H (1 [56] 3))"
      End
      Begin PaneConfiguration = 5
         NumPanes = 2
         Configuration = "(H (2 [66] 3))"
      End
      Begin PaneConfiguration = 6
         NumPanes = 2
         Configuration = "(H (4 [50] 3))"
      End
      Begin PaneConfiguration = 7
         NumPanes = 1
         Configuration = "(V (3))"
      End
      Begin PaneConfiguration = 8
         NumPanes = 3
         Configuration = "(H (1[56] 4[18] 2) )"
      End
      Begin PaneConfiguration = 9
         NumPanes = 2
         Configuration = "(H (1 [75] 4))"
      End
      Begin PaneConfiguration = 10
         NumPanes = 2
         Configuration = "(H (1[66] 2) )"
      End
      Begin PaneConfiguration = 11
         NumPanes = 2
         Configuration = "(H (4 [60] 2))"
      End
      Begin PaneConfiguration = 12
         NumPanes = 1
         Configuration = "(H (1) )"
      End
      Begin PaneConfiguration = 13
         NumPanes = 1
         Configuration = "(V (4))"
      End
      Begin PaneConfiguration = 14
         NumPanes = 1
         Configuration = "(V (2))"
      End
      ActivePaneConfig = 0
   End
   Begin DiagramPane = 
      Begin Origin = 
         Top = 0
         Left = 0
      End
      Begin Tables = 
      End
   End
   Begin SQLPane = 
   End
   Begin DataPane = 
      Begin ParameterDefaults = ""
      End
   End
   Begin CriteriaPane = 
      Begin ColumnWidths = 11
         Column = 1440
         Alias = 900
         Table = 1170
         Output = 720
         Append = 1400
         NewValue = 1170
         SortType = 1350
         SortOrder = 1410
         GroupBy = 1350
         Filter = 1350
         Or = 1350
         Or = 1350
         Or = 1350
      End
   End
End
' , @level0type=N'SCHEMA',@level0name=N'dbo', @level1type=N'VIEW',@level1name=N'TranslatedEvents'
GO
EXEC sys.sp_addextendedproperty @name=N'MS_DiagramPaneCount', @value=1 , @level0type=N'SCHEMA',@level0name=N'dbo', @level1type=N'VIEW',@level1name=N'TranslatedEvents'
GO
EXEC sys.sp_addextendedproperty @name=N'MS_DiagramPane1', @value=N'[0E232FF0-B466-11cf-A24F-00AA00A3EFFF, 1.00]
Begin DesignProperties = 
   Begin PaneConfigurations = 
      Begin PaneConfiguration = 0
         NumPanes = 4
         Configuration = "(H (1[40] 4[20] 2[20] 3) )"
      End
      Begin PaneConfiguration = 1
         NumPanes = 3
         Configuration = "(H (1 [50] 4 [25] 3))"
      End
      Begin PaneConfiguration = 2
         NumPanes = 3
         Configuration = "(H (1 [50] 2 [25] 3))"
      End
      Begin PaneConfiguration = 3
         NumPanes = 3
         Configuration = "(H (4 [30] 2 [40] 3))"
      End
      Begin PaneConfiguration = 4
         NumPanes = 2
         Configuration = "(H (1 [56] 3))"
      End
      Begin PaneConfiguration = 5
         NumPanes = 2
         Configuration = "(H (2 [66] 3))"
      End
      Begin PaneConfiguration = 6
         NumPanes = 2
         Configuration = "(H (4 [50] 3))"
      End
      Begin PaneConfiguration = 7
         NumPanes = 1
         Configuration = "(V (3))"
      End
      Begin PaneConfiguration = 8
         NumPanes = 3
         Configuration = "(H (1[56] 4[18] 2) )"
      End
      Begin PaneConfiguration = 9
         NumPanes = 2
         Configuration = "(H (1 [75] 4))"
      End
      Begin PaneConfiguration = 10
         NumPanes = 2
         Configuration = "(H (1[66] 2) )"
      End
      Begin PaneConfiguration = 11
         NumPanes = 2
         Configuration = "(H (4 [60] 2))"
      End
      Begin PaneConfiguration = 12
         NumPanes = 1
         Configuration = "(H (1) )"
      End
      Begin PaneConfiguration = 13
         NumPanes = 1
         Configuration = "(V (4))"
      End
      Begin PaneConfiguration = 14
         NumPanes = 1
         Configuration = "(V (2))"
      End
      ActivePaneConfig = 0
   End
   Begin DiagramPane = 
      Begin Origin = 
         Top = 0
         Left = 0
      End
      Begin Tables = 
         Begin Table = "Experts"
            Begin Extent = 
               Top = 6
               Left = 38
               Bottom = 427
               Right = 225
            End
            DisplayFlags = 280
            TopColumn = 0
         End
         Begin Table = "Translations"
            Begin Extent = 
               Top = 6
               Left = 263
               Bottom = 136
               Right = 447
            End
            DisplayFlags = 280
            TopColumn = 0
         End
         Begin Table = "Constants"
            Begin Extent = 
               Top = 6
               Left = 485
               Bottom = 320
               Right = 655
            End
            DisplayFlags = 280
            TopColumn = 0
         End
      End
   End
   Begin SQLPane = 
   End
   Begin DataPane = 
      Begin ParameterDefaults = ""
      End
   End
   Begin CriteriaPane = 
      Begin ColumnWidths = 11
         Column = 5655
         Alias = 900
         Table = 1170
         Output = 720
         Append = 1400
         NewValue = 1170
         SortType = 1350
         SortOrder = 1410
         GroupBy = 1350
         Filter = 1350
         Or = 1350
         Or = 1350
         Or = 1350
      End
   End
End
' , @level0type=N'SCHEMA',@level0name=N'dbo', @level1type=N'VIEW',@level1name=N'TranslatedExperts'
GO
EXEC sys.sp_addextendedproperty @name=N'MS_DiagramPaneCount', @value=1 , @level0type=N'SCHEMA',@level0name=N'dbo', @level1type=N'VIEW',@level1name=N'TranslatedExperts'
GO
EXEC sys.sp_addextendedproperty @name=N'MS_DiagramPane1', @value=N'[0E232FF0-B466-11cf-A24F-00AA00A3EFFF, 1.00]
Begin DesignProperties = 
   Begin PaneConfigurations = 
      Begin PaneConfiguration = 0
         NumPanes = 4
         Configuration = "(H (1[40] 4[20] 2[20] 3) )"
      End
      Begin PaneConfiguration = 1
         NumPanes = 3
         Configuration = "(H (1 [50] 4 [25] 3))"
      End
      Begin PaneConfiguration = 2
         NumPanes = 3
         Configuration = "(H (1 [50] 2 [25] 3))"
      End
      Begin PaneConfiguration = 3
         NumPanes = 3
         Configuration = "(H (4 [30] 2 [40] 3))"
      End
      Begin PaneConfiguration = 4
         NumPanes = 2
         Configuration = "(H (1 [56] 3))"
      End
      Begin PaneConfiguration = 5
         NumPanes = 2
         Configuration = "(H (2 [66] 3))"
      End
      Begin PaneConfiguration = 6
         NumPanes = 2
         Configuration = "(H (4 [50] 3))"
      End
      Begin PaneConfiguration = 7
         NumPanes = 1
         Configuration = "(V (3))"
      End
      Begin PaneConfiguration = 8
         NumPanes = 3
         Configuration = "(H (1[56] 4[18] 2) )"
      End
      Begin PaneConfiguration = 9
         NumPanes = 2
         Configuration = "(H (1 [75] 4))"
      End
      Begin PaneConfiguration = 10
         NumPanes = 2
         Configuration = "(H (1[66] 2) )"
      End
      Begin PaneConfiguration = 11
         NumPanes = 2
         Configuration = "(H (4 [60] 2))"
      End
      Begin PaneConfiguration = 12
         NumPanes = 1
         Configuration = "(H (1) )"
      End
      Begin PaneConfiguration = 13
         NumPanes = 1
         Configuration = "(V (4))"
      End
      Begin PaneConfiguration = 14
         NumPanes = 1
         Configuration = "(V (2))"
      End
      ActivePaneConfig = 0
   End
   Begin DiagramPane = 
      Begin Origin = 
         Top = 0
         Left = 0
      End
      Begin Tables = 
         Begin Table = "Menus"
            Begin Extent = 
               Top = 6
               Left = 38
               Bottom = 136
               Right = 211
            End
            DisplayFlags = 280
            TopColumn = 0
         End
         Begin Table = "Translations"
            Begin Extent = 
               Top = 6
               Left = 249
               Bottom = 286
               Right = 433
            End
            DisplayFlags = 280
            TopColumn = 0
         End
      End
   End
   Begin SQLPane = 
   End
   Begin DataPane = 
      Begin ParameterDefaults = ""
      End
   End
   Begin CriteriaPane = 
      Begin ColumnWidths = 11
         Column = 1440
         Alias = 900
         Table = 1170
         Output = 720
         Append = 1400
         NewValue = 1170
         SortType = 1350
         SortOrder = 1410
         GroupBy = 1350
         Filter = 1350
         Or = 1350
         Or = 1350
         Or = 1350
      End
   End
End
' , @level0type=N'SCHEMA',@level0name=N'dbo', @level1type=N'VIEW',@level1name=N'TranslatedMenus'
GO
EXEC sys.sp_addextendedproperty @name=N'MS_DiagramPaneCount', @value=1 , @level0type=N'SCHEMA',@level0name=N'dbo', @level1type=N'VIEW',@level1name=N'TranslatedMenus'
GO
EXEC sys.sp_addextendedproperty @name=N'MS_DiagramPane1', @value=N'[0E232FF0-B466-11cf-A24F-00AA00A3EFFF, 1.00]
Begin DesignProperties = 
   Begin PaneConfigurations = 
      Begin PaneConfiguration = 0
         NumPanes = 4
         Configuration = "(H (1[40] 4[20] 2[20] 3) )"
      End
      Begin PaneConfiguration = 1
         NumPanes = 3
         Configuration = "(H (1 [50] 4 [25] 3))"
      End
      Begin PaneConfiguration = 2
         NumPanes = 3
         Configuration = "(H (1 [50] 2 [25] 3))"
      End
      Begin PaneConfiguration = 3
         NumPanes = 3
         Configuration = "(H (4 [30] 2 [40] 3))"
      End
      Begin PaneConfiguration = 4
         NumPanes = 2
         Configuration = "(H (1 [56] 3))"
      End
      Begin PaneConfiguration = 5
         NumPanes = 2
         Configuration = "(H (2 [66] 3))"
      End
      Begin PaneConfiguration = 6
         NumPanes = 2
         Configuration = "(H (4 [50] 3))"
      End
      Begin PaneConfiguration = 7
         NumPanes = 1
         Configuration = "(V (3))"
      End
      Begin PaneConfiguration = 8
         NumPanes = 3
         Configuration = "(H (1[56] 4[18] 2) )"
      End
      Begin PaneConfiguration = 9
         NumPanes = 2
         Configuration = "(H (1 [75] 4))"
      End
      Begin PaneConfiguration = 10
         NumPanes = 2
         Configuration = "(H (1[66] 2) )"
      End
      Begin PaneConfiguration = 11
         NumPanes = 2
         Configuration = "(H (4 [60] 2))"
      End
      Begin PaneConfiguration = 12
         NumPanes = 1
         Configuration = "(H (1) )"
      End
      Begin PaneConfiguration = 13
         NumPanes = 1
         Configuration = "(V (4))"
      End
      Begin PaneConfiguration = 14
         NumPanes = 1
         Configuration = "(V (2))"
      End
      ActivePaneConfig = 0
   End
   Begin DiagramPane = 
      Begin Origin = 
         Top = -96
         Left = 0
      End
      Begin Tables = 
         Begin Table = "Messages"
            Begin Extent = 
               Top = 6
               Left = 38
               Bottom = 136
               Right = 208
            End
            DisplayFlags = 280
            TopColumn = 0
         End
         Begin Table = "Translations"
            Begin Extent = 
               Top = 6
               Left = 246
               Bottom = 136
               Right = 430
            End
            DisplayFlags = 280
            TopColumn = 0
         End
      End
   End
   Begin SQLPane = 
   End
   Begin DataPane = 
      Begin ParameterDefaults = ""
      End
      Begin ColumnWidths = 9
         Width = 284
         Width = 1500
         Width = 1500
         Width = 1500
         Width = 1500
         Width = 1500
         Width = 1500
         Width = 1500
         Width = 1500
      End
   End
   Begin CriteriaPane = 
      Begin ColumnWidths = 11
         Column = 1440
         Alias = 900
         Table = 1170
         Output = 720
         Append = 1400
         NewValue = 1170
         SortType = 1350
         SortOrder = 1410
         GroupBy = 1350
         Filter = 1350
         Or = 1350
         Or = 1350
         Or = 1350
      End
   End
End
' , @level0type=N'SCHEMA',@level0name=N'dbo', @level1type=N'VIEW',@level1name=N'TranslatedMessages'
GO
EXEC sys.sp_addextendedproperty @name=N'MS_DiagramPaneCount', @value=1 , @level0type=N'SCHEMA',@level0name=N'dbo', @level1type=N'VIEW',@level1name=N'TranslatedMessages'
GO
EXEC sys.sp_addextendedproperty @name=N'MS_DiagramPane1', @value=N'[0E232FF0-B466-11cf-A24F-00AA00A3EFFF, 1.00]
Begin DesignProperties = 
   Begin PaneConfigurations = 
      Begin PaneConfiguration = 0
         NumPanes = 4
         Configuration = "(H (1[40] 4[20] 2[20] 3) )"
      End
      Begin PaneConfiguration = 1
         NumPanes = 3
         Configuration = "(H (1 [50] 4 [25] 3))"
      End
      Begin PaneConfiguration = 2
         NumPanes = 3
         Configuration = "(H (1 [50] 2 [25] 3))"
      End
      Begin PaneConfiguration = 3
         NumPanes = 3
         Configuration = "(H (4 [30] 2 [40] 3))"
      End
      Begin PaneConfiguration = 4
         NumPanes = 2
         Configuration = "(H (1 [56] 3))"
      End
      Begin PaneConfiguration = 5
         NumPanes = 2
         Configuration = "(H (2 [66] 3))"
      End
      Begin PaneConfiguration = 6
         NumPanes = 2
         Configuration = "(H (4 [50] 3))"
      End
      Begin PaneConfiguration = 7
         NumPanes = 1
         Configuration = "(V (3))"
      End
      Begin PaneConfiguration = 8
         NumPanes = 3
         Configuration = "(H (1[56] 4[18] 2) )"
      End
      Begin PaneConfiguration = 9
         NumPanes = 2
         Configuration = "(H (1 [75] 4))"
      End
      Begin PaneConfiguration = 10
         NumPanes = 2
         Configuration = "(H (1[66] 2) )"
      End
      Begin PaneConfiguration = 11
         NumPanes = 2
         Configuration = "(H (4 [60] 2))"
      End
      Begin PaneConfiguration = 12
         NumPanes = 1
         Configuration = "(H (1) )"
      End
      Begin PaneConfiguration = 13
         NumPanes = 1
         Configuration = "(V (4))"
      End
      Begin PaneConfiguration = 14
         NumPanes = 1
         Configuration = "(V (2))"
      End
      ActivePaneConfig = 0
   End
   Begin DiagramPane = 
      Begin Origin = 
         Top = -480
         Left = 0
      End
      Begin Tables = 
      End
   End
   Begin SQLPane = 
   End
   Begin DataPane = 
      Begin ParameterDefaults = ""
      End
   End
   Begin CriteriaPane = 
      Begin ColumnWidths = 11
         Column = 1440
         Alias = 900
         Table = 1170
         Output = 720
         Append = 1400
         NewValue = 1170
         SortType = 1350
         SortOrder = 1410
         GroupBy = 1350
         Filter = 1350
         Or = 1350
         Or = 1350
         Or = 1350
      End
   End
End
' , @level0type=N'SCHEMA',@level0name=N'dbo', @level1type=N'VIEW',@level1name=N'TranslatedNews'
GO
EXEC sys.sp_addextendedproperty @name=N'MS_DiagramPaneCount', @value=1 , @level0type=N'SCHEMA',@level0name=N'dbo', @level1type=N'VIEW',@level1name=N'TranslatedNews'
GO
