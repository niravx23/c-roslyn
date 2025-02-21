using System.Text;
using Microsoft.CodeAnalysis;
using Microsoft.CodeAnalysis.FlowAnalysis;
using Microsoft.CodeAnalysis.MSBuild;

namespace CSharpParser
{
    internal class SourceParser
    {
        public static RulesUtil? rulesUtil;
        public static SemanticModel? SemanticModel;
        public static Compilation? Compilation;
        public static int ErrorsHit = 0;
        public static Solution? Solution;
        public static SyntaxNode CurrentRoot; 

        //List obtained from the parser
        public static List<string>? InvocationExpressionsIdentified; 

        public static async Task Main(string[] args)
        {
            InvocationExpressionsIdentified = new();


            string CsvFilePath = "C:\\Users\\2775556\\Desktop\\report.csv";
            var ListOfInvocationExpressions = await GetInvocations();   // list obtained from the built-in API.
            StringBuilder writer = new StringBuilder();
            writer.AppendLine(string.Format("{4},{0},{1},{2},{3}", "Method Name", "Line Number", "InvocationExpression Captured", "File Path of Source Code", "Sr. No."));
            File.AppendAllText(CsvFilePath, writer.ToString());
            writer.Clear();
            int lineNo = 1;

            foreach (var symbol in InvocationExpressionsIdentified)
            {
         
                //Console.WriteLine(symbol); 
                var parts = symbol.Split('|');
                var newLine = string.Format("{4},{0},{1},{2},{3}", parts[0], parts[1], parts[2], parts[3],lineNo);
                writer.AppendLine(newLine);
                File.AppendAllText(CsvFilePath, writer.ToString());
                writer.Clear();
                lineNo++;
            }

            Console.WriteLine(ErrorsHit);

        }

        private static async Task<List<string>> GetInvocations()
        {
            var   symbols = new List<string>(); 
            using var workspace = MSBuildWorkspace.Create();
            Solution = await workspace.OpenSolutionAsync("C:\\Users\\2775556\\source\\repos\\TesterApp\\TesterApp.sln");
            var   rules  = new RulesUtil();

            List<string> invocationExpressions  =  new List<string>();

            
            foreach (var project in Solution.Projects)
            {
                Compilation = await project.GetCompilationAsync();
                var temp = Compilation.GetDiagnostics();
                
                if (Compilation == null) continue;

                CSharpFileParser fileParser = new CSharpFileParser();

                foreach (var docs in project.Documents)
                {
                    
                    var syntaxTree = await docs.GetSyntaxTreeAsync();
                    if (syntaxTree == null) continue;
                
                    SemanticModel = Compilation.GetSemanticModel(syntaxTree);
                    var root = await syntaxTree.GetRootAsync();
                    CurrentRoot = root; 
                    fileParser.ParseTopLevelStatements(root);
                }

            }

            
            return invocationExpressions ;
        }

    }

    
}





/*
 * 
 * 
 * public const string DiagnosticId = "MyRules0001";

        private static readonly LocalizableString Title = "This is a title";
        private static readonly LocalizableString MessageFormat = "This is a message";
        private static readonly LocalizableString Description = "This is a description";
        private const string Category = "Naming";
        private static readonly DiagnosticDescriptor Rule = new DiagnosticDescriptor(DiagnosticId,
        Title,
        MessageFormat,
        Category,
        DiagnosticSeverity.Error,
        true,
        Description);

        public static void RoslynAnalyzer(MethodDeclarationSyntax context, Document doc, SemanticModel sm,ControlFlowGraph cfg,Compilation c)
        {
            //var res = ValueContentAnalysis.TryGetOrComputeResult
            var statements = context.Body.Statements;
            var dataFlow = SemanticModel.AnalyzeDataFlow(statements.First(),statements.Last());
            var wellKnownTypeProvider = WellKnownTypeProvider.GetOrCreate(c);
            
            foreach (var child in dataFlow.WrittenInside)
            {
                //var cx = ValueContentAnalysis.TryGetOrComputeResult(cfg,,wellKnownTypeProvider,new AnalyzerOptions(new()),Rule,PointsToAnalysisKind.Complete);
            }

        }






    //const string targetPath = "C:\\Users\\2775556\\Desktop\\c#\\TestedApps\\crypthash-net-master\\src\\CryptHash.Net\\CryptHash.Net.sln";
            //var workspace = MSBuildWorkspace.Create();
            //var solution = await workspace.OpenSolutionAsync(targetPath);
            //rulesUtil = new RulesUtil();


            //foreach (var project in solution.Projects)
            //{

            //    foreach (var iterator in project.Documents)
            //    {
            //        if (iterator.SourceCodeKind == SourceCodeKind.Regular)
            //        {
            //            CSharpFileParser parser = new CSharpFileParser(iterator, project);
            //        }
            //    }
            //}
 */



/*
 //var invExp = root.DescendantNodes().OfType<InvocationExpressionSyntax>();

                    //foreach(var exp in invExp)
                    //{
                    //    cnt++; 
                    //    var symbol = SemanticModel.GetSymbolInfo(exp);
                    //    var s = symbol.Symbol as ISymbol;
                    //    var parent = exp.Ancestors().OfType<MethodDeclarationSyntax>().FirstOrDefault();

                    //    if (s == null || parent == null ) continue;

                    //    string res = s.ToDisplayString() + " in " + parent.Identifier + ". Document url - " + docs.FilePath + " Line Number - " + exp.SyntaxTree.GetLineSpan(exp.Span).ToString().Split(" ")[1];
                    //    symbols.Add(res);
                    //}
 */
